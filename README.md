
readme_content = """# 🤖 AI Front Desk Receptionist

An intelligent voice-powered receptionist built with **n8n**, **Vapi**, and **Google Calendar** that handles appointment bookings, availability checks, and business inquiries — 24/7.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Setup Guide](#setup-guide)
- [Workflow Configuration](#workflow-configuration)
- [Vapi Tool Configuration](#vapi-tool-configuration)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## 🎯 Overview

This project automates front desk operations for small businesses (salons, clinics, consultancies) using a voice AI agent. Customers can:

- 📅 **Book appointments** by speaking naturally
- 🔍 **Check availability** for specific dates and times
- ❓ **Ask business questions** (hours, location, services, pricing)

All bookings sync with **Google Calendar** and are logged to **Google Sheets** for record-keeping.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🎙️ **Voice Conversations** | Natural language via phone using Vapi AI |
| 📆 **Smart Booking** | Checks conflicts, business hours, weekends |
| 🕐 **Availability Check** | Real-time calendar lookup with conflict detection |
| 📍 **Business Info** | Answers FAQs about hours, location, services, pricing |
| 📝 **Sheet Logging** | All appointments auto-logged to Google Sheets |
| ⏰ **Business Hours Guard** | Rejects bookings outside 9 AM – 5 PM, Monday–Friday |
| 🗓️ **Weekend Block** | Prevents weekend bookings automatically |
| 🌍 **Timezone Aware** | Handles timezone offsets for accurate scheduling |

---

## 🏗️ Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│   Customer  │────▶│  Vapi AI    │────▶│  n8n Webhooks   │
│  (Phone)    │◀────│  (Voice)    │◀────│  (3 endpoints)  │
└─────────────┘     └─────────────┘     └─────────────────┘
                                                │
                    ┌───────────────────────────┼───────────┐
                    ▼                           ▼           ▼
            ┌──────────────┐          ┌──────────────┐  ┌─────────────┐
            │   Google     │          │   Google     │  │   Google    │
            │   Calendar   │          │   Sheets     │  │   Calendar  │
            │  (Bookings)  │          │  (Logging)   │  │ (Conflict)  │
            └──────────────┘          └──────────────┘  └─────────────┘
```

### Three Webhook Endpoints

| Endpoint | Purpose | Method |
|----------|---------|--------|
| `/webhook/check-availability` | Check if a time slot is free | POST |
| `/webhook/book-appointment` | Book a new appointment | POST |
| `/webhook/business-info` | Answer business questions | POST |

---

## 🛠️ Tech Stack

| Technology | Role |
|------------|------|
 **n8n** | Workflow automation & webhook handling |
| **Vapi** | AI voice agent & phone interface |
| **Google Calendar API** | Appointment scheduling & conflict checking |
| **Google Sheets API** | Appointment logging & record-keeping |
| **ngrok** | Local tunnel for webhook development |

---

## 📦 Prerequisites

Before starting, ensure you have:

- [ ] **n8n** installed (self-hosted or cloud)
- [ ] **Vapi account** with credits
- [ ] **Google Cloud Project** with Calendar & Sheets APIs enabled
- [ ] **ngrok** installed (for local development)
- [ ] **Google OAuth credentials** for Calendar & Sheets access

---

## 🚀 Setup Guide

### 1. Clone & Import Workflow

```bash
# Import the workflow JSON into n8n
# n8n → Workflows → Import from File → Select Voice Agent Tool - Business Info FAQ.json
```

### 2. Configure Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Enable **Google Calendar API** and **Google Sheets API**
3. Create OAuth 2.0 credentials
4. Add credentials to n8n:
   - **Google Calendar OAuth2**
   - **Google Sheets OAuth2**

### 3. Set Up Google Sheet

Create a spreadsheet named **"Appointments"** with these headers:

| Name | Phone | Date | Time | Start Time | End Time | Status | Confirmation ID | Timestamp |
|------|-------|------|------|------------|----------|--------|-----------------|-----------|

### 4. Start ngrok Tunnel

```bash
ngrok http 5678
```

Copy the HTTPS URL (e.g., `https://your-url.ngrok-free.dev`)

### 5. Configure Vapi Tools

In [Vapi Dashboard](https://dashboard.vapi.ai/):

#### Tool 1: `check_availability_tool`

```
Name: check_availability_tool
Description: Use this tool to check if a specific date and time slot is available on the calendar.
Server URL: https://your-ngrok-url/webhook/check-availability
Timeout: 20s

Parameters:
  dateTime (string, required)
  Example: "Today at 3 PM", "Tomorrow at 10 AM", "July 25th at 2 PM"
```

#### Tool 2: `book_appointment_tool`

```
Name: book_appointment_tool
Description: Use this tool to officially book an appointment after confirming details with the user.
Server URL: https://your-ngrok-url/webhook/book-appointment
Timeout: 20s

Parameters:
  name (string, required)
  phone (string, required)
  dateTime (string, required)
  duration (number, optional) — default 30 minutes
```

#### Tool 3: `business_info_tool`

```
Name: business_info_tool
Description: Use this tool to search for general business information like location, operating hours, or pricing.
Server URL: https://your-ngrok-url/webhook/business-info
Timeout: 20s

Parameters:
  topic (string, required)

