# webex_service_app
Webex Service App

This Python script allows you to create a meeting on behalf of a subuser in a Webex organization using [service app](https://developer.webex.com/docs/service-apps) credentials.

## Prerequisites

- Python 3.x
- An active Webex account [developer account](https://developer.webex.com/)
- A regisetered service app

## Dependencies

This script depends on the following Python modules:

- requests
- json
- os
- datetime
- webbrowser

You can install these dependencies using pip:

```bash
pip install requests
```

Note: The other dependencies are part of the Python Standard Library and do not need to be installed separately.

## Setup

1. Replace `YOUR CLIENT ID HERE`, `YOUR CLIENT SECRET HERE`, `ACCESS TOKEN POST ADMIN AUTHORIZATION`, `REFRESH TOKEN POST ADMIN AUTHORIZATION` with your own values. These values are produced by registering a service app on Webex App Hub @ developer.webex.com.

2. Replace `'A sub users email'` with the email of the subuser for whom you want to create the meeting.

## Running the Script

You can run the script using Python:

```bash
python serviceapp.py
```

This will create a meeting 24 hours from the current time, lasting for one hour. If the access token is invalid and returns a 401 error, the script will automatically refresh the tokens and retry creating the meeting.

Remember to store your tokens securely in a production environment.

## License

This project is licensed under the terms of the MIT license.

