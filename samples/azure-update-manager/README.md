# Azure Update Manager Post-Maintenance Webhook

This document describes the JSON payload structure for Azure Update Manager post-maintenance webhooks, with special focus on reboot requirement detection.

## Overview

The Azure Update Manager post-maintenance webhook is triggered after a maintenance window completes, providing detailed information about the updates that were installed, failed, or skipped during the maintenance operation.

## Key Features for Reboot Detection

### Individual Update Reboot Requirements

Each update in the `updates` array includes a `rebootRequired` boolean field:

```json
{
  "updateId": "update-001",
  "title": "Security Update for Windows Server 2022",
  "rebootRequired": true,
  "installationStatus": "Installed"
}
```

### Summary Reboot Status

The `summary` section provides an overall view of reboot requirements:

```json
{
  "summary": {
    "rebootRequired": true,
    "rebootStatus": "Pending",
    "nextMaintenanceWindow": "2024-01-22T10:00:00Z"
  }
}
```

### Reboot Status Values

- `"Pending"` - A reboot is required but has not been performed
- `"Completed"` - A reboot was required and has been completed  
- `"NotRequired"` - No reboot is needed
- `"Failed"` - A reboot was attempted but failed

## Payload Structure

### Root Level Properties

| Field | Type | Description |
|-------|------|-------------|
| `eventType` | string | Always "Microsoft.UpdateManager.PostMaintenance" |
| `eventTime` | string | ISO 8601 timestamp when the event occurred |
| `id` | string | Unique identifier for this event |
| `topic` | string | Azure resource ID of the target resource |
| `data` | object | Main payload containing maintenance details |

### Data Object

| Field | Type | Description |
|-------|------|-------------|
| `resourceId` | string | Full Azure resource ID |
| `status` | string | Overall maintenance status (Completed, Failed, Cancelled) |
| `updates` | array | List of individual updates processed |
| `summary` | object | Aggregated results and reboot information |
| `machine` | object | Target machine information |
| `maintenanceConfiguration` | object | Configuration used for this maintenance |

### Update Object

| Field | Type | Description |
|-------|------|-------------|
| `updateId` | string | Unique identifier for the update |
| `title` | string | Human-readable update title |
| `classification` | string | Update category (SecurityUpdates, CriticalUpdates, etc.) |
| `kbId` | string | Microsoft Knowledge Base ID |
| `installationStatus` | string | Installed, Failed, or Skipped |
| `rebootRequired` | boolean | **Whether this specific update requires a reboot** |
| `errorCode` | string | Error code if installation failed (optional) |
| `errorMessage` | string | Error description if installation failed (optional) |

### Summary Object

| Field | Type | Description |
|-------|------|-------------|
| `totalUpdates` | number | Total number of updates processed |
| `successfulUpdates` | number | Number of successfully installed updates |
| `failedUpdates` | number | Number of failed update installations |
| `rebootRequired` | boolean | **Overall reboot requirement status** |
| `rebootStatus` | string | **Current reboot state (Pending, Completed, NotRequired, Failed)** |
| `nextMaintenanceWindow` | string | Next scheduled maintenance window |

## Usage in Intelligent Applications

This webhook payload can be processed by intelligent applications to:

1. **Automated Reboot Management**: Check the `summary.rebootRequired` field to determine if a system restart is needed
2. **Update Success Analysis**: Use AI to analyze failed updates and recommend remediation actions
3. **Maintenance Reporting**: Generate intelligent reports on patch deployment success rates
4. **Predictive Maintenance**: Use historical webhook data to predict future maintenance windows and requirements

## Example Processing Logic

```python
def process_maintenance_webhook(payload):
    data = payload.get('data', {})
    summary = data.get('summary', {})
    
    # Check if reboot is required
    if summary.get('rebootRequired', False):
        reboot_status = summary.get('rebootStatus', 'Unknown')
        if reboot_status == 'Pending':
            # Schedule reboot or notify administrators
            schedule_reboot(data.get('resourceId'))
        
    # Analyze failed updates
    failed_updates = [update for update in data.get('updates', []) 
                     if update.get('installationStatus') == 'Failed']
    
    if failed_updates:
        analyze_failures(failed_updates)
```

## Related Documentation

- [Azure Update Manager Overview](https://docs.microsoft.com/en-us/azure/update-manager/)
- [Azure Event Grid Integration](https://docs.microsoft.com/en-us/azure/event-grid/)
- [Webhook Security Best Practices](https://docs.microsoft.com/en-us/azure/event-grid/webhook-event-delivery)