# RudderStack Implementation Guide

## Overview

RudderStack is an open-source customer data platform (CDP) that routes events to downstream destinations. Its API is Segment-compatible — the same identify/group/track model — so switching between them requires minimal code changes.

## Installation

**npm (recommended for SPAs):**
```bash
npm install @rudderstack/analytics-js
```

**CDN snippet (traditional websites):**

Add to `<head>`. Replace `WRITE_KEY` and `DATA_PLANE_URL` from your RudderStack source dashboard.

```html
<script type="text/javascript">
!function(){
  var e=window.rudderanalytics=window.rudderanalytics||[];
  e.methods=["load","page","track","identify","alias","group","ready","reset","getAnonymousId","setAnonymousId"];
  e.factory=function(t){return function(){var r=Array.prototype.slice.call(arguments);r.unshift(t);e.push(r);}};
  for(var t=0;t<e.methods.length;t++){var r=e.methods[t];e[r]=e.factory(r);}
  e.loadJS=function(e,t){
    var r=document.createElement("script");
    r.type="text/javascript";r.async=!0;
    r.src="https://cdn.rudderlabs.com/v1.1/rudder-analytics.min.js";
    var a=document.getElementsByTagName("script")[0];
    a.parentNode.insertBefore(r,a);
  };
  e.loadJS();
  e.load("WRITE_KEY", "DATA_PLANE_URL");
}();
</script>
```

## Initialization (npm)

```typescript
import { RudderAnalytics } from '@rudderstack/analytics-js';

const analytics = new RudderAnalytics();
analytics.load('WRITE_KEY', 'DATA_PLANE_URL');
```

## Core Flow: Identify, Group, Track

### 1. Identify the User

Call on login or signup, once the user ID is known.

```typescript
analytics.identify('usr_123', {
  email: 'jane@example.com',
  name: 'Jane Smith',
  role: 'admin',
  plan: 'pro'
});
```

This ties all future events from this browser to the identified user.

### 2. Associate User with a Group

For B2B, always call `group()` after `identify()` to establish account context.

```typescript
analytics.group('acc_456', {
  name: 'Acme Corp',
  plan: 'enterprise',
  employee_count: 50
});
```

For hierarchical products, issue a `group()` call for **every level** in the hierarchy, with `parent_group_id` to establish relationships:

```typescript
// Account (top level)
analytics.group('acc_456', {
  name: 'Acme Corp',
  group_type: 'account',
  plan: 'enterprise'
});

// Workspace (child of account)
analytics.group('ws_789', {
  name: 'Engineering',
  group_type: 'workspace',
  parent_group_id: 'acc_456'
});

// Project (child of workspace)
analytics.group('proj_123', {
  name: 'Q1 Release',
  group_type: 'project',
  parent_group_id: 'ws_789'
});
```

### 3. Track Events

```typescript
analytics.track('report.created', {
  report_id: 'rpt_789',
  report_type: 'standard'
});
```

The SDK automatically attaches the identified user. You do not need to pass `userId` in the browser.

### 4. Reset on Logout

```typescript
analytics.reset();  // Clears userId, traits, and group associations
```

## Group Context on Track Calls

RudderStack's `group()` associates a user with a group, but does **not** automatically attach group context to subsequent `track()` calls. To attribute an event to a specific group, include `groupId` in the track call's `context` object.

```typescript
// Project-level event
analytics.track('task.completed', {
  task_id: 'task_456'
}, {
  context: { groupId: 'proj_123' }
});

// Account-level event
analytics.track('plan.upgraded', {
  from_plan: 'free',
  to_plan: 'pro'
}, {
  context: { groupId: 'acc_456' }
});
```

**Critical limitation:** RudderStack has no native group hierarchy support. The `context.groupId` is what downstream tools (Accoil, Amplitude, Mixpanel) use for event-level group attribution. Hierarchical rollups depend on the downstream tool supporting `parent_group_id` traits on group calls.

## Node.js (Server-Side)

```typescript
import RudderAnalytics from '@rudderstack/rudder-sdk-node';

const analytics = new RudderAnalytics('WRITE_KEY', {
  dataPlaneUrl: 'DATA_PLANE_URL'
});

// Server-side requires userId on every call
analytics.identify({
  userId: 'usr_123',
  traits: {
    email: 'jane@example.com',
    plan: 'pro'
  }
});

analytics.group({
  userId: 'usr_123',
  groupId: 'acc_456',
  traits: {
    name: 'Acme Corp',
    plan: 'enterprise'
  }
});

analytics.track({
  userId: 'usr_123',
  event: 'report.created',
  properties: {
    report_id: 'rpt_789',
    report_type: 'standard'
  },
  context: { groupId: 'acc_456' }
});
```

## Verifying Events

1. Go to your Source in the RudderStack dashboard
2. Open the **Live Events** debugger
3. Trigger actions in your app and confirm `identify`, `group`, and `track` events appear with correct payloads

## Common Pitfalls

1. **Calling track before identify** -- Events are anonymous until identify is called
2. **Forgetting reset on logout** -- Previous user context persists for the next user
3. **Missing group() calls** -- Downstream B2B tools lose account-level attribution
4. **Server-side without userId** -- Unlike the browser SDK, there is no implicit user context
5. **Assuming group context carries to track** -- It does not. Use `context.groupId` on each track call

## Further Documentation

This guide covers the essentials for browser and Node.js integration. For advanced topics (middleware, transformations, device mode vs cloud mode, mobile SDKs, warehouse destinations), see the RudderStack docs at https://www.rudderstack.com/docs/
