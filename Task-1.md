# Task 1 - GTM Event Schema

## 1. Tracking Strategy

Implementing Google Tag Manager (GTM) is central to transforming raw web traffic into actionable marketing intelligence. The objective is to track the complete patient journey from landing page visit to appointment booking. This enables the marketing team to understand user behavior, identify funnel drop-offs, measure campaign performance, and optimize Google Ads based on completed consultations rather than clicks. By routing all event data dynamically through a centralized GTM container, we decouple tracking logic from core application code, ensuring high reliability, fast page loads, and the agility to add or update marketing tags without developer intervention.

---

## 2. Event Schema Table

Below is the comprehensive event schema designed to capture all crucial micro-interactions and macro-conversions across the booking funnel.

| Event | Trigger / Condition | Parameters | Purpose |
| :--- | :--- | :--- | :--- |
| **`booking_started`** | User clicks the "Book Consultation" or main CTA button. | `clinic_location`, `speciality`, `page_location` | Funnel Entry Tracking |
| **`booking_step_completed`** | User successfully submits a step in the multi-step form. | `step_number`, `clinic_location`, `speciality` | Funnel Progression |
| **`clinic_selected`** | User selects a clinic location from the dropdown menu. | `clinic_location` | User Preference Analysis |
| **`speciality_selected`**| User selects a medical speciality from the dropdown. | `speciality` | Demand Analysis |
| **`doctor_selected`** | User chooses a specific physician/doctor. | `doctor_name`, `speciality` | Preference Tracking |
| **`payment_started`** | User submits patient details and initiates payment flow. | `appointment_value`, `currency`, `payment_method` | Funnel Progression |
| **`payment_failed`** | Payment transaction fails during checkout. | `error_code`, `error_message`, `payment_method` | UX Debugging & Drop-off |
| **`appointment_booked`** | Successful appointment confirmation screen load. | `appointment_id`, `clinic_location`, `speciality`, `value`, `currency` | Core Conversion |
| **`form_error`** | Inline validation error triggered on any booking field. | `step_number`, `field_id`, `error_message` | Form Optimization |
| **`faq_toggle`** | User expands or collapses an FAQ accordion item. | `faq_question`, `action` (expand/collapse) | Engagement Tracking |
| **`contact_link_clicked`**| User clicks direct phone, email, or WhatsApp links. | `contact_type` (phone/email/chat), `page_location` | Lead Generation |

---

## 3. Booking Form Tracking

The visual flow of the booking funnel from initial interaction to final confirmation is modeled below:

```mermaid
graph TD
    LP[Landing Page / Service Page]
    -->|Click "Book Consultation"| BS[booking_started]
    -->|Step 1: Location & Speciality Select| S1[Step 1 Completed]
    -->|Step 2: Patient Details Input| S2[Step 2 Completed]
    -->|Step 3: Confirm details / Pay| S3[Step 3 Completed / Payment Started]
    -->|Success / Confirmation| AB[appointment_booked]

    classDef start fill:#f9f,stroke:#333,stroke-width:2px;
    classDef step fill:#bbf,stroke:#333,stroke-width:1px;
    classDef success fill:#bfb,stroke:#333,stroke-width:2px;
    class LP start;
    class S1,S2,S3 step;
    class AB success;
```

---

## 4. dataLayer JSON

To ensure semantic accuracy and seamless integration with GA4 and advertising platforms, the frontend application pushes structured data to the `dataLayer` at each stage.

### Step 1: Selection of Clinic & Speciality
Pushed when the user completes the selection of the clinic location and medical specialty and moves to the next step.

```json
window.dataLayer.push({
  "event": "booking_step_completed",
  "step_number": 1,
  "clinic_location": "Whitefield",
  "speciality": "Orthopaedics"
});
```

### Step 2: Patient Details Input
Pushed when the patient details (e.g., patient type, general age group/demographics if applicable) are successfully validated and submitted.

```json
window.dataLayer.push({
  "event": "booking_step_completed",
  "step_number": 2,
  "clinic_location": "Whitefield",
  "speciality": "Orthopaedics",
  "patient_type": "New Patient",
  "insurance_provider": "Aetna"
});
```

### Step 3: Confirmation and Successful Booking
Pushed immediately upon receipt of the success callback from the appointment API, generating a unique transaction ID and value.

```json
window.dataLayer.push({
  "event": "appointment_booked",
  "appointment_id": "APT-98274-XYZ",
  "clinic_location": "Whitefield",
  "speciality": "Orthopaedics",
  "value": 150.00,
  "currency": "USD"
});
```

---

## 5. GA4 Funnel Exploration

To build and analyze the conversion funnel in Google Analytics 4 (GA4):

1. Navigate to **Explore** -> **Funnel Exploration**.
2. Define the Steps using Custom Events and Parameter criteria:
   - **Step 1: Session Start / Landing Page** (`page_view`)
   - **Step 2: Booking Started** (`booking_started`)
   - **Step 3: Clinic & Speciality Selected** (`booking_step_completed` where `step_number` equals `1`)
   - **Step 4: Patient Details Completed** (`booking_step_completed` where `step_number` equals `2`)
   - **Step 5: Appointment Completed** (`appointment_booked`)
3. **Outcome**: Marketing can easily visualize drop-off rates between steps, allowing targeted optimizations on steps showing high abandonment (e.g., redesigning Step 2 if user details entry is a major friction point).

---

## 6. Google Ads Conversion Tracking

To track conversion value and attribute success directly to search campaigns:

1. **Trigger Configuration**: Create a trigger in GTM that fires only on the custom event `appointment_booked`.
2. **Tag Configuration**: Set up a **Google Ads Conversion Tracking** tag.
3. **Dynamic Parameters**:
   - Map the **Conversion Value** field to a GTM user-defined variable reading the `value` parameter from the `dataLayer` (`150.00`).
   - Map the **Transaction ID** field to a GTM variable reading `appointment_id` (`APT-98274-XYZ`).
   - Map the **Currency Code** to a GTM variable reading `currency` (`USD`).
4. **Attribution & Smart Bidding**: Passing these dynamic values ensures Google Ads Smart Bidding models optimize for high-value clinic appointments rather than raw page views.

---

## 7. Assumptions

- **GTM Snippet Installation**: The GTM container code is properly installed in the `<head>` and `<body>` of all web pages.
- **Persistent State**: Value, clinic, and specialty parameters are preserved in the component state/session memory to be available at the final confirmation step if needed.
- **GA4 Configuration**: The standard GA4 Configuration tag is already running and capturing default page views and user sessions.
- **Data Privacy**: No Personally Identifiable Information (PII) such as patient names, phone numbers, or exact medical diagnoses is pushed to the `dataLayer` or sent to third-party tags.
