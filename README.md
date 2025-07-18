# Webex Simple Service App

[![Python](https://img.shields.io/badge/Python-3.6+-blue.svg)](https://www.python.org/downloads/)
[![License: Cisco Sample Code](https://img.shields.io/badge/License-Cisco%20Sample%20Code-orange.svg)](https://developer.cisco.com/site/license/cisco-sample-code-license/)
[![Webex](https://img.shields.io/badge/Webex-Service%20App-green.svg)](https://developer.webex.com/docs/service-apps)

## Overview

This **Python script** demonstrates how to create a meeting on behalf of a subuser in a Webex organization using [Webex Service App](https://developer.webex.com/docs/service-apps) credentials. It showcases the power of **impersonation** functionality available in Webex Service Apps, allowing administrators to schedule meetings for other users in their organization.

### Key Features

✅ **Meeting Creation**: Create meetings on behalf of organization users  
✅ **Token Management**: Automatic token refresh when access tokens expire  
✅ **Error Handling**: Robust error handling with 401 retry logic  
✅ **Simple Setup**: Minimal configuration required  
✅ **Production Ready**: Includes best practices for token storage considerations

### Use Cases

- **Administrative Tools**: Build tools for admins to schedule meetings for team members
- **Automated Scheduling**: Create meetings programmatically for events or workflows
- **Bulk Operations**: Schedule multiple meetings across different users
- **Integration**: Integrate with existing systems to provide meeting scheduling capabilities

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Code Structure](#code-structure)
- [API Reference](#api-reference)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites

### Required Software
- **Python 3.6+** - [Download Python](https://www.python.org/downloads/)
- **pip** - Python package installer (usually comes with Python)

### Webex Developer Account
- **[Webex Developer Account](https://developer.webex.com/)** - Sign up for free
- **[Webex Service App](https://developer.webex.com/docs/service-apps)** - Registered and authorized by an organization admin

### Required Scopes
Your Webex Service App must be registered with the following scopes:
- **`meeting:admin_schedule_write`** - Required for impersonation functionality (scheduling meetings on behalf of other users)

## Installation

1. **Clone or download this repository**:
   ```bash
   git clone <repository-url>
   cd simple_service_app
   ```

2. **Install dependencies**:
   ```bash
   pip install requests
   ```

   **Note**: Other dependencies (`json`, `os`, `datetime`, `webbrowser`) are part of the Python Standard Library and don't need separate installation.

## Configuration

### Service App Setup

1. **Register a Service App**:
   - Visit [Webex Developer Portal](https://developer.webex.com/)
   - Create a new Service App
   - Select the required scope: `meeting:admin_schedule_write`
   - Note down your **Client ID** and **Client Secret**

2. **Authorize the Service App**:
   - An organization admin must authorize your Service App
   - This will provide you with **Access Token** and **Refresh Token**

### Script Configuration

Edit the `serviceapp.py` file and replace the placeholder values:

```python
# Replace these values with your actual Service App credentials
clientID = "YOUR CLIENT ID HERE"                    # Your Service App Client ID
secretID = "YOUR CLIENT SECRET HERE"                # Your Service App Client Secret
access_token = 'ACCESS TOKEN POST ADMIN AUTHORIZATION'  # Initial access token
refresh_token = 'REFRESH TOKEN POST ADMIN AUTHORIZATION' # Initial refresh token
```

### User Configuration

Replace the placeholder email with the actual subuser's email:

```python
'hostEmail': 'A sub users email'  # Replace with actual user email
```

## Usage

### Basic Usage

Run the script using Python:

```bash
python serviceapp.py
```

### What Happens When You Run It

1. **Meeting Creation**: The script creates a meeting scheduled for **24 hours from now**, lasting **1 hour**
2. **Token Validation**: If the access token is valid, the meeting is created successfully
3. **Automatic Refresh**: If the access token is expired (401 error), the script automatically refreshes the tokens and retries
4. **Success Response**: On success, meeting details are displayed in the console

### Example Output

**Successful Meeting Creation:**
```
statusCode: 200
{
  'id': 'meeting_id_here',
  'title': 'Example Meeting Title',
  'start': '2024-01-15T10:00:00Z',
  'end': '2024-01-15T11:00:00Z',
  'hostEmail': 'user@example.com',
  'webLink': 'https://meet.webex.com/meet/...',
  'meetingNumber': '123456789'
}
```

**Token Refresh (when needed):**
```
function : get_token_refresh()
Token returned in refresh result : eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...
Refresh Token returned in refresh result : eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...
```

## Code Structure

### Main Components

```mermaid
graph TB
    A[Script Start] --> B[create_meeting()]
    B --> C{Access Token Valid?}
    C -->|Yes| D[Meeting Created Successfully]
    C -->|No 401| E[get_tokens_refresh()]
    E --> F[Update Tokens]
    F --> G[Retry create_meeting()]
    G --> D
```

### Functions

#### `create_meeting()`
Creates a meeting on behalf of a subuser in the Webex organization.

**Features:**
- Schedules meeting 24 hours from current time
- Meeting duration: 1 hour
- Uses impersonation via `hostEmail` parameter
- Returns response object with meeting details

**Parameters:**
- Uses global variables for tokens and configuration
- Meeting details are hardcoded but can be customized

**Response:**
- **200**: Meeting created successfully
- **401**: Access token expired (triggers refresh)
- **Other errors**: Various API errors

#### `get_tokens_refresh()`
Refreshes the access token using the refresh token when a 401 error occurs.

**Process:**
1. Makes POST request to `/v1/access_token` endpoint
2. Uses refresh token to get new access and refresh tokens
3. Returns new tokens for immediate use

**Parameters:**
- `grant_type`: "refresh_token"
- `client_id`: Service App Client ID
- `client_secret`: Service App Client Secret
- `refresh_token`: Current refresh token

**Returns:**
- New access token
- New refresh token

### Configuration Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `clientID` | Service App Client ID | `C1234567890abcdef...` |
| `secretID` | Service App Client Secret | `secret123...` |
| `access_token` | Current access token | `Bearer eyJ0eXAiOiJKV1Q...` |
| `refresh_token` | Current refresh token | `eyJ0eXAiOiJKV1QiLCJh...` |

## API Reference

### Webex APIs Used

#### 1. Create Meeting
- **Endpoint**: `POST https://webexapis.com/v1/meetings`
- **Documentation**: [Webex Meetings API](https://developer.webex.com/docs/meetings)
- **Headers**:
  - `Authorization: Bearer {access_token}`
  - `Content-Type: application/json`

**Request Body:**
```json
{
  "title": "Example Meeting Title",
  "start": "2024-01-15T10:00:00Z",
  "end": "2024-01-15T11:00:00Z",
  "hostEmail": "user@example.com"
}
```

#### 2. Refresh Token
- **Endpoint**: `POST https://webexapis.com/v1/access_token`
- **Headers**:
  - `Accept: application/json`
  - `Content-Type: application/x-www-form-urlencoded`

**Request Body:**
```
grant_type=refresh_token&client_id={clientID}&client_secret={secretID}&refresh_token={refresh_token}
```

### Meeting Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | String | Yes | Meeting title (max 128 characters) |
| `start` | String | Yes | Meeting start time (ISO 8601 format) |
| `end` | String | Yes | Meeting end time (ISO 8601 format) |
| `hostEmail` | String | Yes | Email of the user on whose behalf the meeting is created |

### Response Codes

| Code | Description | Action |
|------|-------------|--------|
| 200 | Success | Meeting created successfully |
| 401 | Unauthorized | Access token expired - refresh and retry |
| 400 | Bad Request | Check request parameters |
| 403 | Forbidden | Insufficient permissions or invalid host email |
| 429 | Rate Limited | Implement retry with backoff |

## Error Handling

### Built-in Error Handling

The script includes automatic error handling for common scenarios:

1. **Token Expiration (401)**:
   - Automatically refreshes tokens
   - Retries the meeting creation
   - No manual intervention required

2. **API Errors**:
   - Displays error code and message
   - Provides debugging information

### Custom Error Handling

You can extend the error handling by modifying the `create_meeting()` function:

```python
def create_meeting():
    # ... existing code ...
    
    if response.status_code == 200:
        print('Meeting created successfully!')
        meeting_data = response.json()
        print(f"Meeting ID: {meeting_data.get('id')}")
        print(f"Meeting Link: {meeting_data.get('webLink')}")
    elif response.status_code == 400:
        print('Bad Request: Check your meeting parameters')
    elif response.status_code == 403:
        print('Forbidden: Check host email or permissions')
    elif response.status_code == 429:
        print('Rate Limited: Please wait before retrying')
    else:
        print(f'Error: {response.status_code} - {response.text}')
    
    return response
```

## Best Practices

### Security

1. **Token Storage**: 
   ```python
   # ❌ Don't store tokens in source code (production)
   access_token = 'hardcoded_token'
   
   # ✅ Use environment variables (production)
   import os
   access_token = os.getenv('WEBEX_ACCESS_TOKEN')
   ```

2. **Environment Variables**:
   ```bash
   # Create .env file
   WEBEX_CLIENT_ID=your_client_id
   WEBEX_CLIENT_SECRET=your_client_secret
   WEBEX_ACCESS_TOKEN=your_access_token
   WEBEX_REFRESH_TOKEN=your_refresh_token
   ```

3. **Token Refresh**:
   - Store new tokens after each refresh
   - Implement proper token persistence
   - Use database or secure storage for production

### Code Improvements

1. **Configuration Management**:
   ```python
   import configparser
   
   config = configparser.ConfigParser()
   config.read('config.ini')
   
   clientID = config['webex']['client_id']
   secretID = config['webex']['client_secret']
   ```

2. **Error Logging**:
   ```python
   import logging
   
   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)
   
   def create_meeting():
       try:
           # ... meeting creation logic ...
           logger.info("Meeting created successfully")
       except Exception as e:
           logger.error(f"Failed to create meeting: {e}")
   ```

3. **Input Validation**:
   ```python
   def validate_email(email):
       import re
       pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
       return re.match(pattern, email) is not None
   ```

### Production Considerations

1. **Token Persistence**: Store tokens in a database or secure key vault
2. **Rate Limiting**: Implement retry logic with exponential backoff
3. **Monitoring**: Add logging and monitoring for production use
4. **Configuration**: Use environment variables or config files
5. **Error Handling**: Implement comprehensive error handling and reporting

## Troubleshooting

### Common Issues

#### 1. 401 Unauthorized Error
**Problem**: Access token is expired or invalid.

**Solution**:
- The script automatically handles this by refreshing tokens
- Ensure your refresh token is valid
- Check that your Service App is properly authorized

#### 2. 403 Forbidden Error
**Problem**: Insufficient permissions or invalid host email.

**Solutions**:
- Verify the `hostEmail` belongs to your organization
- Ensure your Service App has `meeting:admin_schedule_write` scope
- Check that the Service App is authorized by an org admin

#### 3. 400 Bad Request Error
**Problem**: Invalid request parameters.

**Solutions**:
- Check meeting start/end times are in ISO 8601 format
- Ensure meeting title is not longer than 128 characters
- Verify all required parameters are provided

#### 4. Import Errors
**Problem**: Module not found errors.

**Solutions**:
- Install required dependencies: `pip install requests`
- Ensure Python 3.6+ is installed
- Check that all imports are correctly spelled

### Debug Mode

Add debug information to troubleshoot issues:

```python
import logging
logging.basicConfig(level=logging.DEBUG)

# Add before API calls
print(f"Using access token: {access_token[:20]}...")
print(f"Meeting data: {json.dumps(body, indent=2)}")
```

### Testing Token Refresh

To test the token refresh functionality, uncomment this line in the script:

```python
# Uncomment this line to force a 401 error and test token refresh
access_token += 'invalid'
```

## Contributing

We welcome contributions to improve this sample code!

### How to Contribute

1. **Fork the repository**
2. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-improvement
   ```
3. **Make your changes**:
   - Follow Python best practices
   - Add comments for complex logic
   - Test your changes thoroughly
4. **Submit a pull request**

### Contribution Guidelines

- **Code Style**: Follow PEP 8 Python style guidelines
- **Documentation**: Update README for new features
- **Testing**: Test your changes with valid Webex credentials
- **Comments**: Add meaningful comments for complex logic

### Improvement Ideas

- Add command-line argument parsing
- Implement configuration file support
- Add bulk meeting creation functionality
- Create a web interface
- Add meeting templates
- Implement scheduled meeting creation

## License

This project is licensed under the **Cisco Sample Code License**.

### License Summary

- ✅ **Permitted**: Copy, modify, and redistribute for use with Cisco products
- ❌ **Prohibited**: Use independent of Cisco products or to compete with Cisco
- ℹ️ **Warranty**: Provided "as is" without warranty
- ℹ️ **Support**: Not supported by Cisco TAC

See the [LICENSE](LICENSE) file for full license terms.

---

## Additional Resources

### Webex Developer Resources
- [Webex Developer Portal](https://developer.webex.com/)
- [Service Apps Documentation](https://developer.webex.com/docs/service-apps)
- [Meetings API Reference](https://developer.webex.com/docs/api/v1/meetings)
- [OAuth 2.0 Documentation](https://developer.webex.com/docs/integrations#oauth-2-0)

### Python Resources
- [Python Official Documentation](https://docs.python.org/3/)
- [Requests Library Documentation](https://requests.readthedocs.io/)
- [JSON Handling in Python](https://docs.python.org/3/library/json.html)

### Community
- [Webex Developer Community](https://developer.webex.com/community)
- [Stack Overflow - Webex](https://stackoverflow.com/questions/tagged/webex)

## Support

For issues and questions:
- Check the [Troubleshooting](#troubleshooting) section
- Review [Webex Developer Documentation](https://developer.webex.com/docs/)
- Visit the [Webex Developer Community](https://developer.webex.com/community)

---

**Happy Coding!** 🚀
