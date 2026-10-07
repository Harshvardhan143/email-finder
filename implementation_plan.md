# Email Finder & Verifier - GitHub Task Tracker

This document breaks down the entire SaaS implementation plan into granular, 1-by-1 actionable tasks. You can use this as a direct reference to create your GitHub Issues or Kanban board tickets.

## Phase 1: Database Setup and Excel Migration
- [x] **Task 1.1: Initialize Project & Repository**
  - Set up Git repository.
  - Create basic folder structure (backend, frontend, scripts).
- [ ] **Task 1.2: Set up PostgreSQL Database**
  - Install PostgreSQL locally or provision a cloud database.
  - Create the primary database `email_finder_db`.
- [ ] **Task 1.3: Design Database Schema**
  - Create `Users` table (id, email, password_hash, role, credits_balance).
  - Create `Companies` table (id, name, domain, industry).
  - Create `Contacts` table (id, first_name, last_name, linkedin_url, verified_email, company_id).
- [ ] **Task 1.4: Build Excel Migration Script**
  - Set up a Python environment (`venv`).
  - Install `pandas` and `psycopg2` / `SQLAlchemy`.
  - Write a script to read the existing Excel data, clean duplicates, and map to the new database tables.
- [ ] **Task 1.5: Execute Migration & Verify Data**
  - Run the script and verify that companies and contacts are correctly populated in PostgreSQL.

## Phase 2: Core Engine (Extraction & Verification)
- [ ] **Task 2.1: Setup Web Scraper Environment**
  - Install Playwright (`pip install playwright`).
  - Configure headless browser settings.
- [ ] **Task 2.2: Build LinkedIn Extraction Script**
  - Write logic to log in and manage session cookies.
  - Write selectors to scrape the "Experience" section and extract the "Present Company".
- [ ] **Task 2.3: Integrate Rotating Proxies (Crucial)**
  - Integrate a proxy provider (e.g., BrightData, Smartproxy) so LinkedIn does not ban your server's IP address.
- [ ] **Task 2.4: Build Database Matching Logic**
  - Write a function that takes the scraped company name and queries the PostgreSQL database for a match in the correct industry.
- [ ] **Task 2.5: Build Email Permutator**
  - Write a Python function that takes `first_name`, `last_name`, and `domain` and returns a list of common email combinations.
- [ ] **Task 2.6: Build SMTP Verifier (Bounce Check)**
  - Write logic to look up DNS MX records for a given domain using `dnspython`.
  - Write the SMTP connection sequence (HELO, MAIL FROM, RCPT TO) using `smtplib`.
  - Add logic to capture `250 OK` (Valid) vs `550` (Invalid).
- [ ] **Task 2.7: Handle "Catch-All" Domains**
  - Update the SMTP verifier to first test a dummy email (e.g., `test12345@domain.com`).
  - Flag the domain as "Catch-All" if the dummy email returns `250 OK`.

## Phase 3: API, Background Tasks & Credit System
- [ ] **Task 3.1: Setup FastAPI Backend**
  - Initialize a FastAPI project and configure CORS.
  - Connect FastAPI to the PostgreSQL database.
- [ ] **Task 3.2: Build User Authentication (JWT)**
  - Create `/register` and `/login` API endpoints.
  - Create middleware to protect private routes.
- [ ] **Task 3.3: Setup Redis & Celery (Background Job Queue)**
  - *Why: Scraping and SMTP checks take time. If 10 users search at once, the server will crash without a queue.*
  - Install Redis (message broker) and Celery (task worker).
  - Move the Scraper and Verifier logic into background Celery tasks.
- [ ] **Task 3.4: Implement the Credit System API**
  - Set default credits (e.g., 50) on user registration.
  - Create logic to deduct 1 credit upon a successful email verification.
- [ ] **Task 3.5: Build the Main Search API Endpoint**
  - Create a `/search` endpoint that triggers the background Celery task.
  - Create a `/status` endpoint for the frontend to poll and see if the background search is finished.

## Phase 4: Frontend UI (User Dashboard)
- [ ] **Task 4.1: Setup Next.js Frontend**
  - Initialize a Next.js (React) project with Tailwind CSS.
- [ ] **Task 4.2: Build Auth Pages**
  - Create Sign Up and Login UIs.
- [ ] **Task 4.3: Build Main Dashboard Layout**
  - Create the Left Sidebar (Navigation) and Top Navbar (Credit Balance).
- [ ] **Task 4.4: Build the Single Search Interface**
  - Create the large central search bar with loading state animations.
- [ ] **Task 4.5: Build Bulk CSV Upload (Highly Requested in SaaS)**
  - Build a drag-and-drop area where recruiters can upload a CSV of 100+ LinkedIn URLs to process in bulk.
- [ ] **Task 4.6: Build Results Table**
  - Create a table to display the extracted name, company, email, and a green/red verification badge. Add an "Export to CSV" button.

## Phase 5: Admin Panel
- [ ] **Task 5.1: Create Admin Role Logic**
  - Add Role-Based Access in the backend (bypasses credit limits).
- [ ] **Task 5.2: Build Admin UI Layout & User Management**
  - Create a protected `/admin` route.
  - Display all users and add a form to manually grant/remove credits.
- [ ] **Task 5.3: Admin Unlimited Bulk Search**
  - Provide an internal search tool in the admin panel for your team to run massive lists without credit deductions.

## Phase 6: Chrome Extension (Future)
- [ ] **Task 6.1: Setup Manifest V3 Boilerplate**
- [ ] **Task 6.2: Build Extension UI & DOM Scraper**
  - Design popup. Write content scripts to read the active LinkedIn tab.
- [ ] **Task 6.3: Connect Extension to API**
  - Pass the authenticated user's JWT from the extension to your backend to process the profile.

## Phase 7: Payments & Monetization (Future)
- [ ] **Task 7.1: Integrate Stripe API**
  - Set up Stripe products (e.g., 500 credits for $29, 2000 credits for $99).
- [ ] **Task 7.2: Build Billing UI**
  - Add a "Buy Credits" page to the user dashboard where users can check out using Stripe Elements.
- [ ] **Task 7.3: Handle Stripe Webhooks**
  - Write a backend endpoint to listen for successful Stripe payments and automatically add credits to the user's balance.
