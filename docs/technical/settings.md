# Settings

## Overview

Consilium provides a settings interface that allows users to customize their experience. This document outlines the available settings and their impact on the system.

## User Interface Settings

### Theme Settings

- **Theme Mode**:
  - Light: Bright theme for daytime use
  - Dark: Dark theme for low-light environments
  - Auto: Automatically switch based on system preferences or time of day

## Web Scraping Settings

- **Maximum Tokens**: Maximum number of tokens to include when scraping web content
  - Default: 100,000
  - Range: 10,000 - 500,000

## Reminder Settings

- **Default Notification Time**: When to send notifications for reminders that don't have a specific time
  - Default: 9:00 AM
  - Options: Customizable time

- **Snooze Duration Options**: Available durations for snoozing reminders
  - Default: ["5 minutes", "30 minutes", "1 hour", "3 hours", "Tomorrow morning"]
  - Note: These are predefined options presented to users, not natural language inputs

## Integration Settings

### ClickUp Integration

- **Account Connection**: Connect/disconnect ClickUp account
  - Status: Connected/Disconnected
  - Last synced timestamp

- **Workspace Selection**: Select which ClickUp workspaces to sync
  - Multi-select from available workspaces

- **Sync Frequency**: How often to sync with ClickUp
  - Options: Real-time, Hourly, Daily, Manual only

### Email Integration

- **Email Account**: Connect/disconnect email account
  - Status: Connected/Disconnected

- **Task Creation from Email**: Enable creating tasks from emails
  - Options: On/Off

- **Email Notifications**: Receive email notifications for tasks
  - Options: All tasks, High priority only, Off
