#Known Issues for Version 1811

**Users cannot log on to Citrix Receiver for Web if a custom authentication form contains an element with `ID=confirmBtn`**

Users are unable to log on to Citrix Receiver for Web if a StoreFront authentication extension generates a custom authentication form containing an element with ID `confirmBtn`.<br>
Workaround: The authentication extension should use a different ID value in the custom form. [603196]


