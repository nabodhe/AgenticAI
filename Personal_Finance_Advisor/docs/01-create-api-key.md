# Chapter 1: Create a Google API Key

> **Navigation:** [← Back to Index](../README.md) | [Table of Contents](../README.md) | [Next: Chapter 2: Create a Python Virtual Environment →](./02-virtual-environment.md)

## Overview

Before building any agents with Google's Agent Development Kit (ADK), you need a valid **Google API Key** with the **Gemini API** enabled.

> **Security Note:** Never commit your actual API key to Git. Store it in a local `.env` file and ensure `.env` is listed in your `.gitignore`.

---

## Step-by-Step Guide

### 1. Open Google Cloud Console
Navigate to [Google Cloud Console](https://console.cloud.google.com/).

### 2. Create a New Project
1. Click **Select Project** → **New Project**.
2. Enter a descriptive project name (e.g., `personal-finance-advisor-ai`).
3. Click **Create**.

### 3. Enable Required APIs
1. In the left sidebar, navigate to **APIs & Services** → **Library**.
2. Search for and enable the **Gemini API** *(required)*.
3. *(Optional)* If extending your project with location features, you can also enable:
   - Google Maps API
   - Geocoding API
   - Places API

### 4. Generate an API Key
1. Navigate to **APIs & Services** → **Credentials**.
2. Click **Create Credentials** → **API Key**.
3. Copy the generated key and save it securely.

### 5. (Recommended) Restrict the API Key
1. Click on your newly created API key in the **Credentials** list.
2. Under **API Restrictions**, select **Restrict key** and allow only **Generative Language API (Gemini API)**.
3. Click **Save**.

---

> **Navigation:** [← Back to Index](../README.md) | [Table of Contents](../README.md) | [Next: Chapter 2: Create a Python Virtual Environment →](./02-virtual-environment.md)
