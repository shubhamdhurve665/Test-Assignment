# EDMO Enrollment Intelligence Integration

## Overview

This solution implements the Salesforce-side integration for EDMO (Enrollment Intelligence Platform). The application receives enrollment readiness scores from an external system, stores them in Salesforce, updates Contact enrollment priority, and displays the latest score on the Contact record page.

---

## Components Implemented

### 1. Apex REST API

**Class:** `EnrollmentScoreAPI`

Features:

* Exposes a REST endpoint:
  `/services/apexrest/enrollment-scores/`
* Accepts batch enrollment score requests.
* Looks up Contacts using email address.
* Creates `Enrollment_Score__c` records for valid Contacts.
* Returns per-record success/failure responses.
* Supports partial success processing.
* Uses `with sharing` to enforce record-level security.

---

### 2. Trigger Framework

**Trigger:** `EnrollmentScoreTrigger`

**Handler:** `EnrollmentScoreTriggerHandler`

Features:

* Updates `Enrollment_Priority__c` on Contact.
* Score Mapping:

  * 75–100 → Hot
  * 40–74 → Warm
  * 0–39 → Cold
* Fully bulkified.
* No SOQL or DML inside loops.
* Handles multiple scores for the same Contact.
* Latest `Scored_At__c` value determines final priority.

---

### 3. Lightning Web Component

**Component:** `enrollmentScoreCard`

Features:

* Displays latest enrollment score information.
* Displays:

  * Score
  * Recommended Action
  * Source System
  * Scored Date
* Uses `@wire` with Apex controller.
* Displays message when no score exists.
* Designed for Contact Record Pages.

---

### 4. Apex Controller

**Class:** `EnrollmentScoreController`

Features:

* Returns latest Enrollment Score for a Contact.
* Uses `@AuraEnabled(cacheable=true)`.
* Enforces object accessibility checks.
* Supports reactive LWC updates.

---

### 5. Test Class

**Class:** `EnrollmentAssignmentTest`

Test Coverage Includes:

* REST API positive scenario.
* REST API Contact not found scenario.
* Bulk processing scenario.
* Contact priority update verification.
* Latest score wins validation.

Target coverage achieved through meaningful assertions and business logic validation.

---

## Design Decisions

1. Used bulkified processing to support batch requests.
2. Used Trigger Handler Pattern to separate trigger and business logic.
3. Implemented partial success handling for REST integration.
4. Enforced security using `with sharing`.
5. Implemented FLS/Object accessibility checks in Apex controller.
6. Used latest `Scored_At__c` timestamp to determine Contact priority.
7. Designed reusable and maintainable code structure.

---

## Deployment Steps

1. Deploy metadata to Salesforce Org.
2. Create Custom Object:

   * Enrollment_Score__c
3. Create Custom Fields:

   * Contact__c
   * Score__c
   * Recommended_Action__c
   * Source_System__c
   * Scored_At__c
4. Create Contact Field:

   * Enrollment_Priority__c
5. Deploy Apex Classes, Trigger, and LWC.
6. Add `enrollmentScoreCard` component to Contact Record Page.
7. Run Apex Tests.

---

## Sample REST Request

```json
{
  "scores": [
    {
      "contactEmail": "john@test.com",
      "score": 85,
      "recommendedAction": "Schedule Call",
      "sourceSystem": "EDMO Bot v2",
      "scoredAt": "2026-03-10T10:30:00Z"
    }
  ]
}
```

## Expected Result

* Enrollment Score record created.
* Contact Priority updated.
* Latest score displayed on Contact record page.
* Success response returned to calling system.
