# Browser Automation Patterns

## Identity iframe context — antixss header
```javascript
// antixss cookie is readable from Identity iframe context
const antixss = document.cookie.match(/antixss=([^;]+)/)?.[1];
// Use in headers: {'Content-Type': 'application/json', 'antixss': antixss}
```

## Privilege Cloud context — XSRF token
```javascript
// XSRF-TOKEN cookie is readable from pcloud tab
const xsrf = document.cookie.match(/XSRF-TOKEN-[^=]+=([^;]+)/)?.[1];
// Use in headers: {'Content-Type': 'application/json', 'X-XSRF-TOKEN': xsrf}
```

## ExtJS Grid Interaction (Identity admin uses ExtJS 4.2.3)
```javascript
Ext.getCmp('window-1961')                                      // Find component by ID
Ext.getCmp('window-1961').query('button,textfield').map(c => c.id)  // Query children
var btn = Ext.getCmp('button-1919'); btn.handler.call(btn.scope||btn, btn)  // Click button
Ext.getCmp('jsutil-text-1944').fireEvent('specialkey', cmp, {getKey: ()=>13})  // Enter key
Ext.getCmp('grid-id').getStore().getAt(0)                      // Virtualized grid access
Ext.WindowManager.getActive().id                               // Active window
```
