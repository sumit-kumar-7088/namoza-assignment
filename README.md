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
(We will fill this out later)


### Step 2: Paste and Save
1. Paste all of that copied text directly into your empty `README.md` file in VS Code.
2. Press `Ctrl + S` on your keyboard to save the file.

### Step 3: Save to Git (Commit)
Now we need to tell Git to permanently save this step in your project history. 
1. Go back to your terminal window (which should still be open to your `namoza-assignment` folder).
2. Type `git add .` (make sure to include the space and the period) and press **Enter**. This tells Git to prepare all files for saving.
3. Type `git commit -m "Added Task 1 GTM Schema"` and press **Enter**. This officially saves the snapshot of your work.