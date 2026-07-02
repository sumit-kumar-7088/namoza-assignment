Markdown
# Namoza Developer Assignment - Position 1

## Task 01 - GTM Event Schema

| Event Name | Trigger Type | Key Parameters (DataLayer variables) | GA4 Report / Audience Destination |
| :--- | :--- | :--- | :--- |
| `booking_step_complete` | Custom Event | `step_number`, `step_name`, `clinic_location` | Funnel Exploration, Audience: "Booking Abandoners" |
| `call_button_click` | Click - Just Links (URL contains `tel:`) | `link_url`, `page_location`, `button_location` | Standard Events Report, Conversions |
| `whatsapp_chat_initiated`| Click - Just Links (URL contains `wa.me`) | `link_url`, `page_location`, `clinic_location` | Standard Events Report, Conversions |
| `patient_guide_download` | Custom Event (Gated form submit) | `file_name`, `file_extension`, `user_phone_provided` | Standard Events Report, Audience: "High Intent Patients" |
| `clinic_location_view` | Page View / DOM Ready | `page_location`, `clinic_city`, `clinic_neighborhood` | Pages and Screens Report |
| `blog_article_scroll` | Scroll Depth (50%, 75%, 90%) | `percent_scrolled`, `page_title`, `article_category` | Engagement > Pages and Screens Report |

### 3-Step Booking Form Funnel Architecture

**Step 1: Location & Specialty Selected**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee Replacement"
}
Step 2: Patient Details Entered

JSON
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "patient_details_entered",
  "preferred_date": "2023-11-20",
  "phone_prefix": "+91",
  "lead_type": "new_patient"
}
Step 3: Booking Confirmed

JSON
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee Replacement",
  "appointment_id": "ORD-99823"
}
Surfacing Step-Level Drop-Off in GA4:
Use the GA4 Funnel Exploration report matching the booking_step_complete event and filtering by step_number 1, 2, and 3 sequentially to create a drop-off waterfall.

Google Ads Conversion Action
Chosen Action: booking_step_complete where step_number = 3 (Booking Confirmed).
Justification: This represents a guaranteed bottom-of-funnel acquisition, unlike chat clicks which only indicate intent. Optimizing bidding algorithms for actual bookings prevents wasted ad spend.

Task 03 - Integration Design
### 1. End-to-End Architecture
To ensure scalability, reliability, and proper data handling, I would architect this integration using a custom Webhook connected to an automation middleware like **Make (formerly Integromat)**, paired with Google Tag Manager on the front-end. 

**The Flow:**
1. **Front-End / GTM:** Upon form submission, the vanilla JS immediately pushes the `consultation_form_submitted` event to the dataLayer. GTM listens for this and fires the Google Ads Conversion tag directly in the browser. 
2. **Data Payload:** Simultaneously, the JS makes an asynchronous `POST` request to a Make Webhook URL containing the Name, Phone, and Clinic Preference.
3. **Middleware (Make.com):** 
   - **Step A (The HubSpot Trap):** HubSpot natively deduplicates by *Email*, not Phone. Since we only collect Phone numbers, a native form embed or standard Zapier push will result in massive duplication issues. In Make, I would first use the HubSpot `Search CRM` module to search for the Phone Number. 
   - **Step B:** If the contact exists, Make uses the `Update a Contact` module. If not, it uses `Create a Contact`, mapping the fields (Source = 'Google Ads...', Lead Status = 'New Enquiry').
   - **Step C:** Make then triggers an HTTP module to make a POST request to the Karix WhatsApp Business API to dispatch the confirmation message.

*Justification:* I chose Make over a direct API call or native embed because it visually handles the crucial conditional logic required for phone-based deduplication without writing heavy back-end code, and allows for robust error-handling paths.

### 2. Single Biggest Failure Point & Fallback
The biggest failure point is the initial Webhook connection—if Make's webhook is down, or the user's connection drops right after clicking submit, the lead is lost entirely. 
**Fallback:** I would implement a `localStorage` fallback in the front-end JS. If the `fetch` request to the webhook fails (catches an error), the lead data is saved in the browser's local storage. A background script can routinely check `localStorage` and retry the webhook submission when the connection is restored. 

### 3. WhatsApp 2-Minute SLA
**What could break it:** Make processing queues during high traffic, Karix API rate limits, or delayed webhook triggers from HubSpot if we relied on HubSpot workflows instead of Make. 
**Monitoring:** I would monitor this by passing a `submit_timestamp` from the front-end through to Make. I would configure a Karix delivery webhook to send delivery receipts back to a secondary Make scenario. This scenario calculates the difference between `submit_timestamp` and `delivery_timestamp`. If the delta exceeds 120 seconds, it instantly fires a Slack alert to the engineering team.

