

# Unity Mobile Notifications Package

**Supported features:**

- Schedule local repeatable or one-time notifications
- Cancel scheduled or already displayed notifications
-  Android specific:
  - Create and modify notification channels/categories on Android Oreo and above
  - Set custom notification icons.

- iOS specific:
  - Use the Apple Push Notification Service  (APNs) to send remote notifications.
  - Modify remote notification content if they are received when the app is running.
  - Group notification into threads (iOS 12 only)



**Requirements:**

- Supports Android 4.1 (API 16)/iOS 10 and above.
- Compatible with Unity 2018.2 and above.

## Summary:

The runtime API is split into two classes AndroidNotificationManager and iOSNotificationManager which respectively can be used to schedule local notification for Android and iOS respectively.

TODO

## Examples:

**Android:**

- **Create a notification channel:**

  Every local notification must belong to a notificatiuon channel, notification channels are only supported by the system on Android Oreo (9.0) and above. On previous versions the channel behaviour is simultate, therefore settings such as priority  (`Importance`) should be set on the channel.

  ```
  var c = new AndroidNotificationChannel()
  {
      Id = "channel_id",
      Name = "Default Channel",
      Importance = Importance.High,
      Description = "Generic notifications",
  }
  AndroidNotificationManager.RegisterNotficationChannel(c);
  ```

- **Send a simple notification:**

  This example show how to schedule a simple notification with some text in it.

  ```
  
  var notification = new AndroidNotification();
  notificationTitle = "SomeTitle";
  notification.Text = "SomeText";
  notification.FireTime = System.DateTime.Now.AddMinutes(5);
  
  //You should specify a custom icon for each notification otherwise a default Unity 
  //icon will be shown in the status bar when instead. You can configure notification 
  // icons in "Edit -> Project Settings -> Mobile Notification Settings".
  notification.Icon = "my_custom_icon_id";
  
  //Optionally you can also set a large icon which will be shown in notification view in // place of the small icon (which will be placed in a small badge atop the large icon)
  notification.LargeIcon = "my_custom_large_icon_id"
  
  // When scheduling each notification is assigned an unique identifier number
  // which can later be used to track notifications status or cancel it.
  var identifier = AndroidNotificationManager.SendNotification(n, "channel_id");
  
  //You can check if the notification was already delivered and perform an action
  //depending on the result. However notification status can only be tracked on Android //Marshmallow (6.0)  and anove.
  if ( CheckScheduledNotificationStatus(identifier) == NotificationStatus.Scheduled)
  {
  	// Replace the currently sheduled notification with a new notification.
  	UpdateScheduledNotifcation(identifier, newNotification);
  }
  else if ( CheckScheduledNotificationStatus(identifier) == NotificationStatus.Delivered)
  {
  	//Remove the notification from the status bar
  	CancelNotification(identifier)
  }
  else if ( CheckScheduledNotificationStatus(identifier) == NotificationStatus.Unknown)
  {
      var identifier = AndroidNotificationManager.SendNotification(n, "channel_id");
  }
  
  
  ```

- **Handling received notifications when the app is running**

  You can subscribe to the *AndroidNotificationManager.OnNotificationReceived* even to receive callbacks whenever a notification is delivered if the app is running.

  ```
  AndroidNotificationManager.OnNotificationReceived +=(int identifier, AndroidNotification notification, string channel)
  {
  	var msg = "Notification received : " + identifier + "\n";
  	msg += "\n Notification received: ";
  	msg += "\n .Title: " + notification.Title;
  	msg += "\n .Body: " + notification.Text;
  	msg += "\n .Channel: " + channel.Name;
  	Debug.Log(msg);
  };
  ```



**iOS:**

- **Request authorization:**

  Request the system for permission to post local notification and receive remote notification. After completion you can retrieve the retrieve the *DeviceToken* ig you created the requests with *registerForRemoteNotifications* set to true and the app succesful registered on the APN. Which can be ued to send push notifications to the device. See [Apple Developer Site](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1) on how to add push notification support to your app. 



  The user migh only grants permission for certain notification features (in this case we requested the permission to show UI Alert dialogs and show a badge on the app icon) in this case *RequestAuthorizationRequest.Granted* would still be *True* and you can check the actual authorization status by *iOSNotificationManager.GetNotificationSettings*. 



  Alternatively you can enable *Request Authorization on App Launch* in *Edit -> Project Settings -> Mobile Notification Settings* then the app will automatically request for authorization when it's launched. Afterwards you might call this method again to determine the current authorization status but the UI system prompt will not be shown again if the user has already granted or denied authorization for this app.



  ```
  using (var req = new RequestAuthorizationRequest(AuthorizationOption.AuthorizationOptionAlert | AuthorizationOption.AuthorizationOptionBadge, true))
  {
  	while (!req.IsFinished)
  	{
  		yield return null;
  	};
  
  	string res = "\n RequestAuthorization: \n";
  	res += "\n finished: " + req.IsFinished;
  	res += "\n granted :  " + req.Granted;
  	res += "\n error:  " + req.Error;
  	res += "\n deviceToken:  " + req.DeviceToken;
  	Debug.Log(res);
  }
  
  ```

- **To Send a Simple and Cancel Notification in X seconds:**

  ```
  var timeTrigger = new iOSNotificationTimeIntervalTrigger()
  {
  	TimeInterval = new TimeSpan(0, minutes, seconds),
  	Repeats = false
  };
  				
  var notification = new iOSNotification()
  {
  	// You can optional specify a custom Identifier which can later be 
  	// used to cancel the notification, if you don't an unique string 
  	// will be generated automtically.
  	//Identifier = "_notification_01" 
  	Title = title,
  	Body = "Scheduled at: " + DateTime.Now.ToShortDateString() + " triggered in 5 seconds",
  	Subtitle = "This is a subtitle, something, something important...",
  	ShowInForeground = true,
  	ForegroundPresentationOption = foregoungOption,
  	CategoryIdentifier = "category_a",
  	ThreadIdentifier = thread,
  	Trigger = timeTrigger,
  };
  		
  iOSNotificationManager.ScheduleNotification(notification);
  
  // You can cancel the notification if wasn't yet triggered:
  iOSNotificationManager.RemoveScheduledNotification(notification.Identifier);
  
  //If the notification was already displayed to the user, you can remove it from the 
  //Notification Center:
  iOSNotificationManager.RemoveDeliveredNotification(notification.Identifier)
  
  ```

- **Other triggers:**

  Besides the time interval trigger you can also use calendar and location trigger:

  ```
  //All the fields in iOSNotificationCalendarTrigger are optional but you need to set 
  //atleast one for it work. For example if you only set the hour and minute fields the  //system will automitcally trigger the notification on the next specified hour and minite.
  var calendarTrigger = new iOSNotificationCalendarTrigger()
  {
  	// Year = 2018,
  	// Month = 8,
  	//Day = 30,
  	Hour = 12,
  	Minute = 0,
  	// Second = 0
  	Repeats = false
  };
  
  // You can also create location trigger when you want to schedule the delivery of a 
  // notification when the device enters or leaves a specific geographic region.
  // Before scheduling any notifications using this trigger,
  // your app must have authorization to use Core Location and must have when-in-use 
  // permissions. Use the Unity LocationService API to request for this authorization.
  // See https://developer.apple.com/documentation/corelocation/clregion?language=objc for // additional information.
  var locationTrigger = iOSNotificationLocationTrigger()
  {
  	// The center coordinate is defined using the WGS 84 system.
  	// In this case the notification would be triggered if the user
  	// entered an area within a 250 meter radius around Eiffel Tower in Paris.
  	Vector2 Center = new Vector2(2.294498, 48.858263),
  	Radius = 250,
  	NotifyOnEntry = true,
  	NotifyOnExit = false,
  }
  
  ```


- **Handling received notifications when the app is running **

  You might want to perform a custom action instead of just showing a notification alert if it's triggered while the app is running:

  ```
  // By default if a local notification is triggered while the app that scheduled it is
  // in the foreground an alert will not be shown for that notification.
  // If you wish the notification to behave the same way as if the the app was not running
  // you need to enable this `ShowInForeground` when scheduling the notification:"
  
  notification.ShowInForeground = True
  
  // In this case you will also need to specify it's 'ForegroundPresentationOption'
  notification.ForegroundPresentationOption = (PresentationOption.NotificationPresentationOptionSound | PresentationOption.NotificationPresentationOptionAlert)
  
  // Alternatively you might wish to perform some other action, like displaying the 
  // notification content using the in-game UI, when the notification is triggered.
  // In this case you need to subscribe to the `OnNotificationReceived` which will be 
  // called whenever a local or a remote notification is received (irregardless if it's shown in the foregound).
  
  iOSNotificationManager.OnNotificationReceived += notification =>
  {
  	var msg = "Notification received : " + notification.Identifier + "\n";
  	msg += "\n Notification received: ";
  	msg += "\n .Title: " + notification.Title;
  	msg += "\n .Badge: " + notification.Badge;
  	msg += "\n .Body: " + notification.Body;
  	msg += "\n .CategoryIdentifier: " + notification.CategoryIdentifier;
  	msg += "\n .Subtitle: " + notification.Subtitle;
  	Debug.Log(msg);
  };
  
  // When receiving remote notification while the app is running you might wish modify the // remote notification content or not show it at all. You can do this by subscribing 
  // to the `OnRemoteNotificationReceived` event. Please note that if you do this remote
  // notifications will never be displayed when the app is running. If you still wish to 
  // show an alert for it you'll have to schedule a local notification using the remote 
  // notifications content:
  
  iOSNotificationManager.OnRemoteNotificationReceived += notification =>
  {
  	// When a remote notification is received modify it's contents and show it
      // after 1 second.
  	var timeTrigger = new iOSNotificationTimeIntervalTrigger()
  	{
  		TimeInterval = new TimeSpan(0, 0, 1),
  		Repeats = false
  	};
  				
  	iOSNotification  n = new iOSNotification()
  	{
  		Title = "Remote : " + notification.Title,
  		Body =  "Remote : " + notification.Body,
  		Subtitle =  "RERemote: " + notification.Subtitle,
  		ShowInForeground = true,
  		ForegroundPresentationOption = PresentationOption.NotificationPresentationOptionSound | PresentationOption.NotificationPresentationOptionAlert | PresentationOption.NotificationPresentationOptionBadge,
  		CategoryIdentifier = notification.CategoryIdentifier,
  		ThreadIdentifier = notification.ThreadIdentifier,
  		Trigger = timeTrigger,
  	};
  	iOSNotificationManager.ScheduleNotification(n);
  			
  	Debug.Log("Rescheduled remote notifications with id: " + notification.Identifier);
  
  };
  ```



## FAQ

{{Nobody asked any questions | Please ask some questions }}

## 