# 🌩️ Supabase Setup Guide for MediaHoard

This guide will help you set up the backend infrastructure required for MediaHoard's cloud synchronization features (Profiles, Settings, Themes, etc.).

## 1. Create a Supabase Project

1.  Go to [supabase.com](https://supabase.com) and sign in.
2.  Click **"New Project"**.
3.  Choose your Organization, Name (e.g., "MediaHoard"), and Region.
4.  Set a Database Password and click **"Create new project"**.
5.  Wait a few minutes for the project to finish provisioning.

## 2. Database Schema Setup

Once your project is ready, you need to create the tables. The easiest way is to use the **SQL Editor**.

1.  In your Supabase dashboard, go to the **SQL Editor** (icon on the left sidebar looking like a terminal `>_`).
2.  Click **"New Query"**.
3.  Copy and paste the following SQL script into the editor:

```sql
-- 1. Create Accounts Table
-- Stores user credentials (custom auth system)
CREATE TABLE public.accounts (
    id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
    created_at timestamp with time zone DEFAULT now(),
    username text NOT NULL UNIQUE,
    secret_hash text NOT NULL
);

-- 2. Create Profiles Table
-- Stores individual user profiles associated with an account
CREATE TABLE public.profiles (
    id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
    created_at timestamp with time zone DEFAULT now(),
    account_id uuid REFERENCES public.accounts(id) ON DELETE CASCADE,
    name text NOT NULL,
    icon text,
    color text
);

-- 3. Create Profile_Data Table
-- Stores the actual data (addons, lists, settings) for each profile as JSON blobs
CREATE TABLE public.profile_data (
    profile_id uuid REFERENCES public.profiles(id) ON DELETE CASCADE,
    key text NOT NULL,
    value jsonb,
    updated_at timestamp with time zone DEFAULT now(),
    PRIMARY KEY (profile_id, key)
);

-- 4. Enable Row Level Security (RLS) - Optional for personal use but recommended
-- For a personal instance where you have the Anon Key, you generally need to allow access.
-- Since this app uses a custom auth implementation on top of public tables (secured by app-side hashing),
-- we will enable RLS but allow public access for simplicity in this setup.
-- Ideally, you would move to Supabase Auth for stricter RLS.

ALTER TABLE public.accounts ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.profile_data ENABLE ROW LEVEL SECURITY;

-- Allow all operations for now (You rely on the app's secret phrase for security)
CREATE POLICY "Enable all access for accounts" ON public.accounts FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Enable all access for profiles" ON public.profiles FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Enable all access for profile_data" ON public.profile_data FOR ALL USING (true) WITH CHECK (true);
```

4.  Click **"Run"** to execute the script. You should see "Success" in the results area.

## 3. Connect MediaHoard to Supabase

Now you need to link your application to your new database.

1.  In Supabase, go to **Project Settings** (gear icon) -> **API**.
2.  Find the **Project URL** and the **anon** / **public** Key.
3.  Open MediaHoard.
4.  Click the **Settings (Gear)** icon in the sidebar.
5.  Scroll to the **Cloud / Supabase** section.
6.  Enter your **Project URL** and **Anon Key**.
7.  Click **Save**.

## 4. Verification

1.  In MediaHoard settings, click **"Login / Connect"**.
2.  Choose the **"Create Account"** tab.
3.  Enter a username and a "Secret Phrase" (this acts as your password).
4.  Click **Create Account**.
5.  If successful, you will remain logged in, and your data will begin syncing!
