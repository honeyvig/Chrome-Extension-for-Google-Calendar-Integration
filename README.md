# Chrome-Extension-for-Google-Calendar-Integration
create a Chrome extension that allows users to easily add and schedule websites as events in Google Calendar. The extension should provide a user-friendly interface, enabling seamless integration with the Google Calendar API. Key functionalities include selecting a website, setting the event details, and scheduling. If you have experience with Chrome extension development and Google API, we would love to hear from you.

The chrome extension should allow other users besides me to register and download it so it will need registration/authentication 
-------
Creating a Chrome extension that integrates with Google Calendar API, allowing users to add and schedule websites as events, involves several key steps. We'll walk through how to structure the extension, including user authentication via Google OAuth, event creation on Google Calendar, and a user interface to manage the scheduling.
Key Features:

    User Authentication using Google OAuth.
    Website Selection for the event (this can be the current website URL or user-provided).
    Event Creation: Create events on Google Calendar with the selected website and event details.
    Scheduling: Allow the user to set a date and time for the event.
    Registration and Download: Users should be able to register and download the extension.

Steps:

    Set Up Google API and OAuth:
        You will need to set up OAuth credentials in the Google Cloud Console to integrate Google Calendar API.

    Chrome Extension Development:
        Manifest file: Define the extension, permissions, and required API.
        Background script: Handle OAuth authentication and communicate with Google Calendar.
        Content script: Interact with the page (to grab the website URL).
        Popup: A user-friendly interface for setting up events.

1. Setting Up Google OAuth and API Access:

Go to the Google Developer Console, create a new project, and enable the Google Calendar API.

    OAuth 2.0 Credentials:
        Go to APIs & Services > Credentials.
        Create OAuth 2.0 Client IDs.
        Set the redirect URI to https://<YOUR_EXTENSION_ID>.chromiumapp.org/ (for development).
        Save the Client ID and Client Secret.

2. Manifest File (manifest.json):

This file defines the extension settings, permissions, and scripts.

{
  "manifest_version": 3,
  "name": "Website to Google Calendar",
  "description": "Easily add and schedule websites as events in Google Calendar.",
  "version": "1.0",
  "permissions": [
    "identity",
    "https://www.googleapis.com/*",
    "storage",
    "activeTab"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "images/icon16.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  },
  "oauth2": {
    "client_id": "YOUR_GOOGLE_CLIENT_ID",
    "scopes": ["https://www.googleapis.com/auth/calendar"]
  },
  "host_permissions": [
    "https://*.google.com/",
    "https://*.googleusercontent.com/"
  ]
}

3. Background Script (background.js):

This will handle the Google OAuth authentication and interact with the Google Calendar API.

// background.js

chrome.runtime.onInstalled.addListener(() => {
  console.log('Website to Google Calendar extension installed');
});

function getAuthToken() {
  return new Promise((resolve, reject) => {
    chrome.identity.getAuthToken({interactive: true}, (token) => {
      if (chrome.runtime.lastError || !token) {
        reject(new Error('Authorization failed'));
      } else {
        resolve(token);
      }
    });
  });
}

async function createCalendarEvent(eventDetails) {
  try {
    const token = await getAuthToken();
    const eventData = {
      summary: eventDetails.summary,
      location: eventDetails.location,
      description: eventDetails.description,
      start: {
        dateTime: eventDetails.start,
        timeZone: 'America/Los_Angeles'
      },
      end: {
        dateTime: eventDetails.end,
        timeZone: 'America/Los_Angeles'
      }
    };
    const response = await fetch('https://www.googleapis.com/calendar/v3/calendars/primary/events', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(eventData)
    });

    const result = await response.json();
    if (result.id) {
      alert('Event created successfully');
    } else {
      alert('Failed to create event');
    }
  } catch (error) {
    console.error('Error creating event: ', error);
  }
}

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'createEvent') {
    createCalendarEvent(message.eventDetails);
  }
});

4. Popup HTML (popup.html):

This file provides the user interface for the extension where users can set the event details.

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Add to Google Calendar</title>
    <style>
      body { font-family: Arial, sans-serif; }
      label { display: block; margin: 5px 0; }
      input, button { padding: 10px; width: 100%; }
    </style>
  </head>
  <body>
    <h1>Schedule Website Event</h1>
    <label for="event-title">Event Title</label>
    <input type="text" id="event-title" placeholder="Enter event title" />
    
    <label for="event-url">Website URL</label>
    <input type="text" id="event-url" placeholder="Enter website URL" />
    
    <label for="event-date">Event Date</label>
    <input type="datetime-local" id="event-date" />
    
    <button id="schedule-event">Add to Calendar</button>
    
    <script src="popup.js"></script>
  </body>
</html>

5. Popup Script (popup.js):

This handles user input, retrieves the website URL, and sends a message to the background script to create the calendar event.

// popup.js

document.getElementById('schedule-event').addEventListener('click', () => {
  const eventTitle = document.getElementById('event-title').value;
  const eventUrl = document.getElementById('event-url').value;
  const eventDate = document.getElementById('event-date').value;

  if (eventTitle && eventUrl && eventDate) {
    const eventDetails = {
      summary: eventTitle,
      location: eventUrl,
      description: `Visit the website: ${eventUrl}`,
      start: new Date(eventDate).toISOString(),
      end: new Date(new Date(eventDate).getTime() + 60 * 60 * 1000).toISOString() // 1-hour event duration
    };

    chrome.runtime.sendMessage({
      type: 'createEvent',
      eventDetails: eventDetails
    });
  } else {
    alert('Please fill all fields');
  }
});

6. Authentication Flow:

    The extension uses Google's identity API to authenticate the user and fetch a Google Calendar token.
    The user will be prompted to grant the extension permissions to access their Google Calendar when they first try to add an event.

7. Testing the Extension:

    Go to chrome://extensions/ and enable Developer Mode.
    Click Load unpacked and select the extension folder.
    Test it by selecting a website and scheduling an event.
    Ensure the event appears on your Google Calendar.

8. Deployment and Registration:

To make the extension available for download by other users:

    Upload the extension to the Chrome Web Store.
    Add a registration mechanism if necessary, which may involve user signup via a backend server if you want to collect user data.
