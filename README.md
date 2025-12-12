# SailPoint IdentityNow Disable Account Action

Disable an account in SailPoint IdentityNow, temporarily revoking access for users. This action creates an asynchronous provisioning task to disable the specified account.

## Overview

This SGNL action integrates with SailPoint IdentityNow to disable user accounts. When executed, it calls the `/v3/accounts/{id}/disable` API endpoint to initiate an account disable operation that processes asynchronously in SailPoint.

## Prerequisites

- SailPoint IdentityNow tenant with API access enabled
- API authentication credentials (supports 4 auth methods - see Configuration below)
- Account ID of the account to disable
- Appropriate permissions to manage account lifecycle

## Configuration

### Authentication

This action supports four authentication methods. Configure one of the following:

#### Option 1: Bearer Token (SailPoint API Token)
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `BEARER_AUTH_TOKEN` | Secret | Yes | Bearer token for SailPoint IdentityNow API authentication |

#### Option 2: Basic Authentication
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `BASIC_USERNAME` | Secret | Yes | Username for SailPoint IdentityNow authentication |
| `BASIC_PASSWORD` | Secret | Yes | Password for SailPoint IdentityNow authentication |

#### Option 3: OAuth2 Client Credentials
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `OAUTH2_CLIENT_CREDENTIALS_CLIENT_SECRET` | Secret | Yes | OAuth2 client secret |
| `OAUTH2_CLIENT_CREDENTIALS_CLIENT_ID` | Environment | Yes | OAuth2 client ID |
| `OAUTH2_CLIENT_CREDENTIALS_TOKEN_URL` | Environment | Yes | OAuth2 token endpoint URL |
| `OAUTH2_CLIENT_CREDENTIALS_SCOPE` | Environment | No | OAuth2 scope |
| `OAUTH2_CLIENT_CREDENTIALS_AUDIENCE` | Environment | No | OAuth2 audience |
| `OAUTH2_CLIENT_CREDENTIALS_AUTH_STYLE` | Environment | No | OAuth2 auth style |

#### Option 4: OAuth2 Authorization Code
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `OAUTH2_AUTHORIZATION_CODE_ACCESS_TOKEN` | Secret | Yes | OAuth2 access token |

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `ADDRESS` | Yes | Default SailPoint IdentityNow API base URL | `https://example.api.identitynow.com` |

### Input Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `accountId` | string | Yes | The ID of the account to disable | `2c91808570bbdc7f0170c02c3a6301f5` |
| `address` | string | No | Override API base URL | `https://custom.api.identitynow.com` |
| `externalVerificationId` | string | No | External verification ID for the disable operation | `ticket-12345` |
| `forceProvisioning` | boolean | No | Force provisioning of the account disable operation | `true` |

### Output Structure

| Field | Type | Description |
|-------|------|-------------|
| `accountId` | string | The ID of the account that was disabled |
| `disabled` | boolean | Whether the account was successfully disabled |
| `taskId` | string | The ID of the provisioning task created |
| `message` | string | Additional information about the operation |
| `disabledAt` | datetime | When the operation was initiated (ISO 8601) |
| `address` | string | The SailPoint IdentityNow API base URL used |

## Usage Example

### Job Request

```json
{
  "id": "disable-account-001",
  "type": "nodejs-22",
  "script": {
    "repository": "github.com/sgnl-actions/sailpoint-identity-now-disable-account",
    "version": "v1.0.0",
    "type": "nodejs"
  },
  "script_inputs": {
    "accountId": "2c91808570bbdc7f0170c02c3a6301f5",
    "externalVerificationId": "ticket-12345",
    "forceProvisioning": true
  },
  "environment": {
    "ADDRESS": "https://example.api.identitynow.com"
  }
}
```

### Successful Response

```json
{
  "accountId": "2c91808570bbdc7f0170c02c3a6301f5",
  "disabled": true,
  "taskId": "2c91808770c3ee34017a6302ec9b03ac",
  "message": "Account disable operation initiated",
  "disabledAt": "2024-01-15T10:30:00Z"
}
```

## How It Works

The action performs a POST request to the SailPoint IdentityNow API to disable the account:

1. **Validate Input**: Ensures accountId parameter is provided
2. **Authenticate**: Uses configured authentication method to get authorization
3. **Build Request**: Constructs the request with optional parameters
4. **Disable Account**: Makes POST request to `/v3/accounts/{accountId}/disable`
5. **Return Task**: Returns the provisioning task ID for tracking

**Note**: Account disable operations in SailPoint are asynchronous. The API returns immediately with a task ID, and the actual disable operation processes in the background.

## Error Handling

The action includes error handling with automatic retry logic managed by the framework:

### HTTP Status Codes
- **202 Accepted**: Successful disable request (expected response)
- **400 Bad Request**: Invalid account ID or account state
- **401 Unauthorized**: Invalid authentication credentials
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Account not found
- **429 Rate Limit**: Too many requests
- **502/503/504 Service Errors**: Temporary service issues

### Automatic Retry Logic

The framework automatically handles retry logic for transient errors. The error handler simply re-throws errors, allowing the framework to determine retry behavior based on the error type and status code.

## Development

### Local Testing

```bash
# Install dependencies
npm install

# Run tests
npm test

# Test locally with mock data
npm run dev

# Build for production
npm run build
```

### Running Tests

The action includes comprehensive unit tests covering:
- Input validation (accountId parameter)
- Authentication handling (all 4 auth methods)
- Success scenarios with optional parameters
- Error handling (API errors, missing credentials)
- Error handler behavior (re-throwing errors)
- Account ID URL encoding

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Check test coverage
npm run test:coverage
```

## Security Considerations

- **Credential Protection**: Never log or expose authentication credentials
- **User Impact**: Disabling accounts immediately revokes access
- **Audit Logging**: All operations are logged in SailPoint's audit system with tracking IDs
- **Input Validation**: Account IDs are URL-encoded to prevent injection attacks
- **Permissions**: Ensure API credentials have only necessary permissions
- **Token Security**: Store API tokens securely, use short-lived tokens when possible

## SailPoint API Reference

This action uses the following SailPoint IdentityNow API endpoint:
- [Disable Account](https://developer.sailpoint.com/docs/api/v3/disable-account/) - POST `/v3/accounts/{id}/disable`
- [Authentication](https://developer.sailpoint.com/docs/api/authentication/)

## Troubleshooting

### Common Issues

1. **"Invalid or missing accountId parameter"**
   - Ensure the `accountId` parameter is provided and is a non-empty string
   - Verify the account ID exists in your SailPoint tenant

2. **"No URL specified. Provide address parameter or ADDRESS environment variable"**
   - Set the ADDRESS environment variable or provide address parameter
   - Example: `https://example.api.identitynow.com`

3. **"No authentication configured"**
   - Ensure you have configured one of the four supported authentication methods
   - Check that the required secrets/environment variables are set

4. **"Failed to disable account: HTTP 404"**
   - Verify the account ID is correct
   - Check that the account exists in SailPoint IdentityNow
   - Ensure you're using the correct tenant URL

5. **"Failed to disable account: HTTP 403"**
   - Ensure your API credentials have permission to disable accounts
   - Check SailPoint admin console for required permissions
   - Verify source system permissions

6. **"Failed to disable account: HTTP 400"**
   - Account may already be disabled
   - Check account status in SailPoint UI
   - Verify the account state allows disable operation

7. **Account not actually disabled after success response**
   - Disable operations are asynchronous
   - Check the provisioning task status using the returned task ID
   - Monitor SailPoint provisioning queue in the UI
   - Verify source system connectivity

### Debug Information

When troubleshooting issues:
- Check the returned `taskId` in SailPoint UI for provisioning status
- Review the `trackingId` from error messages for support cases
- Monitor SailPoint logs for detailed error information
- Verify source system is reachable and responding
- Check source system-specific disable requirements


## License

MIT

## Support

For issues or questions:
- Check SailPoint IdentityNow API documentation
- Review the provisioning task status in SailPoint UI using the returned task ID
- Contact SailPoint support with the tracking ID from error messages
- Create an issue in this repository
