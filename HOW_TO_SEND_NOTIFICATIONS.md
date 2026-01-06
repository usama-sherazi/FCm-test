# How to Send FCM Notifications from Firebase

There are several ways to send Firebase Cloud Messaging (FCM) notifications to your Flutter app. Here are the most common methods:

## Method 1: Using Firebase Console (Easiest - For Testing)

This is the simplest way to send test notifications.

### Steps:

1. **Go to Firebase Console**
   - Visit [https://console.firebase.google.com/](https://console.firebase.google.com/)
   - Select your project (`fcmtest-da4e3`)

2. **Navigate to Cloud Messaging**
   - In the left sidebar, click **"Build"** → **"Cloud Messaging"**
   - Or go directly to: `https://console.firebase.google.com/project/YOUR_PROJECT_ID/notification`

3. **Send Your First Message**
   - Click **"Send your first message"** or **"New notification"**
   - Fill in the form:
     - **Notification title**: e.g., "Test Notification"
     - **Notification text**: e.g., "This is a test message from Firebase"
     - **Notification image** (optional): Upload an image
   
4. **Target Your App**
   - Choose **"Send test message"**
   - Paste your **FCM Registration Token** (copy it from your app's UI)
   - Click **"Test"**

5. **Or Send to All Users**
   - Choose **"Send to topic"**
   - Enter topic name: `all` (your app is subscribed to this topic)
   - Click **"Next"** → **"Review"** → **"Publish"**

### What Happens:
- **When app is in foreground**: Message appears in your app's UI
- **When app is in background**: Notification appears in system tray
- **When app is terminated**: Notification appears in system tray

---

## Method 2: Using FCM HTTP v1 API (For Production)

Use this method when you need to send notifications programmatically from your server.

### Prerequisites:
1. Get your **Server Key** or **Service Account**:
   - Go to Firebase Console → Project Settings → Cloud Messaging
   - Copy the **Server Key** (Legacy) or create a **Service Account** (Recommended)

### Using cURL (Quick Test):

```bash
curl -X POST https://fcm.googleapis.com/v1/projects/YOUR_PROJECT_ID/messages:send \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "token": "YOUR_FCM_REGISTRATION_TOKEN",
      "notification": {
        "title": "Test Notification",
        "body": "This is a test message"
      },
      "data": {
        "FCM": "https://firebase.google.com/docs/cloud-messaging",
        "flutter": "https://flutter.dev/"
      }
    }
  }'
```

### Using Node.js Example:

```javascript
const admin = require('firebase-admin');
const serviceAccount = require('./path/to/serviceAccountKey.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

const message = {
  notification: {
    title: 'Test Notification',
    body: 'This is a test message from Node.js'
  },
  data: {
    FCM: 'https://firebase.google.com/docs/cloud-messaging',
    flutter: 'https://flutter.dev/'
  },
  token: 'YOUR_FCM_REGISTRATION_TOKEN'
};

admin.messaging().send(message)
  .then((response) => {
    console.log('Successfully sent message:', response);
  })
  .catch((error) => {
    console.log('Error sending message:', error);
  });
```

### Using Python Example:

```python
import requests
import json

url = "https://fcm.googleapis.com/v1/projects/YOUR_PROJECT_ID/messages:send"
headers = {
    "Authorization": "Bearer YOUR_ACCESS_TOKEN",
    "Content-Type": "application/json"
}

data = {
    "message": {
        "token": "YOUR_FCM_REGISTRATION_TOKEN",
        "notification": {
            "title": "Test Notification",
            "body": "This is a test message from Python"
        },
        "data": {
            "FCM": "https://firebase.google.com/docs/cloud-messaging",
            "flutter": "https://flutter.dev/"
        }
    }
}

response = requests.post(url, headers=headers, data=json.dumps(data))
print(response.json())
```

---

## Method 3: Send to Topic (Broadcast to Multiple Users)

Your app is already subscribed to the topic `all`. You can send messages to all subscribed devices:

### Using Firebase Console:
1. Go to Cloud Messaging → New notification
2. Choose **"Send to topic"**
3. Enter topic: `all`
4. Fill in title and body
5. Click **"Publish"**

### Using HTTP API:

```bash
curl -X POST https://fcm.googleapis.com/v1/projects/YOUR_PROJECT_ID/messages:send \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "topic": "all",
      "notification": {
        "title": "Broadcast Message",
        "body": "This message goes to all subscribed devices"
      }
    }
  }'
```

---

## Method 4: Using Postman (For Testing API)

1. **Setup Request**:
   - Method: `POST`
   - URL: `https://fcm.googleapis.com/v1/projects/YOUR_PROJECT_ID/messages:send`
   - Headers:
     - `Authorization`: `Bearer YOUR_ACCESS_TOKEN`
     - `Content-Type`: `application/json`

2. **Body (raw JSON)**:
```json
{
  "message": {
    "token": "YOUR_FCM_REGISTRATION_TOKEN",
    "notification": {
      "title": "Test from Postman",
      "body": "This is a test notification"
    },
    "data": {
      "custom_key": "custom_value"
    }
  }
}
```

---

## Getting Your FCM Registration Token

1. **Run your Flutter app**
2. **Look at the app UI** - The token is displayed at the top
3. **Or check the console logs** - The token is printed when the app starts
4. **Copy the token** - It's a long string that looks like:
   ```
   dKx8Y9zQ...very-long-string...xyz123
   ```

---

## Message Types

### 1. Notification Message (Shows in System Tray)
```json
{
  "message": {
    "token": "YOUR_TOKEN",
    "notification": {
      "title": "Hello",
      "body": "World"
    }
  }
}
```

### 2. Data Message (Handled by Your App)
```json
{
  "message": {
    "token": "YOUR_TOKEN",
    "data": {
      "key1": "value1",
      "key2": "value2"
    }
  }
}
```

### 3. Combined (Notification + Data)
```json
{
  "message": {
    "token": "YOUR_TOKEN",
    "notification": {
      "title": "Hello",
      "body": "World"
    },
    "data": {
      "custom_key": "custom_value"
    }
  }
}
```

---

## Testing Checklist

- [ ] App is running and showing FCM token
- [ ] Copy the token from app UI
- [ ] Send test message from Firebase Console
- [ ] Verify message appears in app (foreground)
- [ ] Put app in background and send another message
- [ ] Verify notification appears in system tray
- [ ] Test topic messaging (send to topic "all")

---

## Troubleshooting

### Token Not Working?
- Make sure you copied the entire token (it's very long)
- Tokens can change - get a fresh one if needed
- Check that your app has internet connection

### Not Receiving Messages?
- Check app permissions (should be granted)
- Verify Firebase project is correct
- Check console logs for errors
- Make sure `google-services.json` is in the right place

### Background Messages Not Showing?
- Check Android manifest permissions
- Verify notification channel is set up
- Test on a physical device (emulators sometimes have issues)

---

## Resources

- [Firebase Console](https://console.firebase.google.com/)
- [FCM Documentation](https://firebase.google.com/docs/cloud-messaging)
- [FCM HTTP v1 API Reference](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)
- [FlutterFire FCM Guide](https://firebase.flutter.dev/docs/messaging/overview)

