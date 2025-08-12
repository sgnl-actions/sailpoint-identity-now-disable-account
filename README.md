# SailPoint IdentityNow Disable Account Action

This SGNL action disables an account in SailPoint IdentityNow. It's commonly used to temporarily revoke access for users who should not have access to systems.

## Overview

The action calls the SailPoint IdentityNow `/v3/accounts/{id}/disable` API endpoint to initiate an account disable operation. This creates a provisioning task that processes asynchronously in SailPoint.

## Prerequisites

- SailPoint IdentityNow tenant with API access disabled
- Valid API credentials (Personal Access Token or OAuth2)
- Account ID of the account to disable
- Appropriate permissions to disable accounts

## Configuration

### Secrets

| Name | Description | Required |
|------|-------------|----------|
| `SAILPOINT_API_TOKEN` | SailPoint IdentityNow API token (Bearer token or OAuth2 access token) | Yes |

### Environment Variables

| Name | Description | Default |
|------|-------------|---------|
| `RATE_LIMIT_BACKOFF_MS` | Milliseconds to wait when rate limited | 30000 |
| `SERVICE_ERROR_BACKOFF_MS` | Milliseconds to wait on service errors | 10000 |

### Input Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `accountId` | string | The ID of the account to disable | Yes |
| `sailpointDomain` | string | SailPoint IdentityNow tenant domain (e.g., `example.api.identitynow.com`) | Yes |
| `externalVerificationId` | string | External verification ID for the disable operation | No |
| `forceProvisioning` | boolean | Force provisioning of the account disable operation | No |

### Output Parameters

| Name | Type | Description |
|------|------|-------------|
| `accountId` | string | The ID of the account that was disabled |
| `disabled` | boolean | Whether the account was successfully disabled |
| `taskId` | string | The ID of the provisioning task created |
| `message` | string | Additional information about the operation |
| `disabledAt` | datetime | When the account was disabled (ISO 8601) |

## Usage Examples

### Basic Disable
```json
{
  "accountId": "2c91808570bbdc7f0170c02c3a6301f5",
  "sailpointDomain": "example.api.identitynow.com"
}
```

### Disable with Verification
```json
{
  "accountId": "2c91808570bbdc7f0170c02c3a6301f5",
  "sailpointDomain": "example.api.identitynow.com",
  "externalVerificationId": "ticket-12345",
  "forceProvisioning": true
}
```

## Error Handling

The action includes automatic retry logic for common transient errors:

### Retryable Errors
- **429 Rate Limit**: Waits 30 seconds (configurable) before retrying
- **502 Bad Gateway**: Waits 10 seconds before retrying  
- **503 Service Unavailable**: Waits 10 seconds before retrying
- **504 Gateway Timeout**: Waits 10 seconds before retrying

### Fatal Errors
- **400 Bad Request**: Invalid parameters or account state
- **401 Unauthorized**: Invalid or expired credentials
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Account ID does not exist

## Security Considerations

- **API Token Security**: Store API tokens securely in the secrets manager, never in code
- **Input Validation**: Account IDs are URL-encoded to prevent injection attacks
- **Audit Trail**: All disable operations are logged in SailPoint's audit system
- **Permissions**: Ensure the API token has only the necessary permissions

## Troubleshooting

### Common Issues

1. **Account Not Found**
   - Verify the account ID is correct
   - Check that the account exists in the specified tenant

2. **Permission Denied**
   - Verify API token has permission to disable accounts
   - Check source system permissions

3. **Already Disabled**
   - The account may already be in a disabled state
   - Check account status before attempting disable

4. **Provisioning Delays**
   - Disable operations are asynchronous
   - Check the returned task ID for status
   - Monitor SailPoint provisioning queue

### Debug Information

Disable debug logging by checking:
- SailPoint API response details in logs
- Task ID for tracking in SailPoint UI
- Tracking ID for support cases

## Development

### Running Tests
```bash
npm test
npm run test:coverage
```

### Building
```bash
npm run build
```

### Linting
```bash
npm run lint
npm run lint:fix
```

## API Reference

- [SailPoint IdentityNow API - Disable Account](https://developer.sailpoint.com/docs/api/v3/disable-account/)
- [Authentication](https://developer.sailpoint.com/docs/api/authentication/)

## Support

For issues or questions:
- Check SailPoint IdentityNow documentation
- Review the task status in SailPoint UI using the returned task ID
- Contact SailPoint support with the tracking ID from error messages