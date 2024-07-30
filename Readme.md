# Combine toolbars from different addins in ArcGIS Pro.

This demo shows an example of how you can have multiple addins in ArcGIS Pro and combine their toolbars into a single toolbar. Each addin can be used separately with its own tab and toolbar, but if the 'master' is installed then it will combine them onto that.

## Notes

**Important note. When making changes to the config.daml I find it is important to do a 'Clean' then a build otherwise the changes don't always take effect.**

The master project has no specific code to make this function, all code is in 'secondary'.

In secondary there are two files which control the functionality.

Config.daml

This has the following dependencies and conditions:

```
	<dependencies>
		<!--Force it to load afterout master toolbar addin-->
		<!-- don't worry it won't fail if it doesn't exist -->
		<dependency name="{265229e4-ac53-4818-9e25-8c0a730d6d04}" />
	</dependencies>

	<conditions>
		<insertCondition id="Is_Master_Installed_Installed_Condition">
			<and>
				<state id="Master_Not_Installed" />
			</and>
		</insertCondition>
	</conditions>
```


The dependency is master addin to ensure that this loads after (so it can find it reliably in the module1.cs). The second is a condition which is used to hide this toolbar if the master is installed.

Next ensure auto load is set on the secondary module

```
 <insertModule id="ProCombinedToolbarsSecondary_Module" className="Module1" autoLoad="true" caption="Module1">
 ```

 Add the condition to our tab to hide it if the master is installed

 ```
   <tabs>
    <tab id="ProCombinedToolbarsSecondary_Tab1" caption="Combine Secondary" condition="Is_Master_Installed_Installed_Condition">
      <group refID="ProCombinedToolbarsSecondary_Group1" />
    </tab>
  </tabs>
  ```

Next update the master toolbar to include the secondary toolbar

```
 <!--Now we add it to our master toolbar-->
 <updateModule refID="ProCombinedToolbarsMaster_Module">
  <tabs>
	  <updateTab refID="ProCombinedToolbarsMaster_Tab1">
		  <!--insert any groups from this addin-->
		  <insertGroup refID="ProCombinedToolbarsSecondary_Group1" insert="after" />  
	  </updateTab>
  </tabs>
 </updateModule>
 ```


Module1.cs is then used to set the condition

```
 public Module1()
 {
     // add the state with otherwise the tab still turns up
     FrameworkApplication.State.Activate("Master_Not_Installed");

     var m = FrameworkApplication.FindModule("ProCombinedToolbarsMaster_Module");
     if (m == null)
     {
         FrameworkApplication.State.Activate("Master_Not_Installed");
     }
     else
     {
         FrameworkApplication.State.Deactivate("Master_Not_Installed"); //deactivates the state
     }

 }

```