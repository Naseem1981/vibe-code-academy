# ClearCRM — Database Schema Reference

Complete schema for all 5 ClearCRM tables. Run these in your Supabase SQL Editor in order.

---

## 1. Create Tables

```sql
-- ============================================================
-- TABLE: profiles
-- Extends Supabase auth.users with display info
-- ============================================================
CREATE TABLE profiles (
  id         UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name  TEXT,
  avatar_url TEXT,
  role       TEXT NOT NULL DEFAULT 'member' CHECK (role IN ('admin', 'member')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- TABLE: companies
-- ============================================================
CREATE TABLE companies (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name        TEXT NOT NULL,
  domain      TEXT,
  industry    TEXT,
  website     TEXT,
  phone       TEXT,
  address     TEXT,
  notes       TEXT,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- TABLE: contacts
-- ============================================================
CREATE TABLE contacts (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  company_id  UUID REFERENCES companies(id) ON DELETE SET NULL,
  first_name  TEXT NOT NULL,
  last_name   TEXT,
  email       TEXT,
  phone       TEXT,
  job_title   TEXT,
  notes       TEXT,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- TABLE: deals
-- ============================================================
CREATE TABLE deals (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  contact_id  UUID REFERENCES contacts(id) ON DELETE SET NULL,
  company_id  UUID REFERENCES companies(id) ON DELETE SET NULL,
  title       TEXT NOT NULL,
  value       NUMERIC(12, 2) DEFAULT 0,
  stage       TEXT NOT NULL DEFAULT 'lead'
              CHECK (stage IN ('lead', 'qualified', 'proposal', 'negotiation', 'closed-won', 'closed-lost')),
  close_date  DATE,
  notes       TEXT,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- TABLE: activities
-- ============================================================
CREATE TABLE activities (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  contact_id  UUID REFERENCES contacts(id) ON DELETE CASCADE,
  deal_id     UUID REFERENCES deals(id) ON DELETE CASCADE,
  company_id  UUID REFERENCES companies(id) ON DELETE CASCADE,
  type        TEXT NOT NULL DEFAULT 'note'
              CHECK (type IN ('note', 'call', 'email', 'meeting', 'task')),
  subject     TEXT NOT NULL,
  body        TEXT,
  due_date    TIMESTAMPTZ,
  completed   BOOLEAN NOT NULL DEFAULT FALSE,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 2. Enable Row Level Security

```sql
ALTER TABLE profiles   ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies  ENABLE ROW LEVEL SECURITY;
ALTER TABLE contacts   ENABLE ROW LEVEL SECURITY;
ALTER TABLE deals      ENABLE ROW LEVEL SECURITY;
ALTER TABLE activities ENABLE ROW LEVEL SECURITY;
```

---

## 3. RLS Policies

```sql
-- ============================================================
-- PROFILES policies
-- ============================================================
CREATE POLICY "Users can view their own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);

CREATE POLICY "Users can insert their own profile"
  ON profiles FOR INSERT
  WITH CHECK (auth.uid() = id);

-- ============================================================
-- COMPANIES policies
-- ============================================================
CREATE POLICY "Users can view their own companies"
  ON companies FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own companies"
  ON companies FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own companies"
  ON companies FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own companies"
  ON companies FOR DELETE
  USING (auth.uid() = user_id);

-- ============================================================
-- CONTACTS policies
-- ============================================================
CREATE POLICY "Users can view their own contacts"
  ON contacts FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own contacts"
  ON contacts FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own contacts"
  ON contacts FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own contacts"
  ON contacts FOR DELETE
  USING (auth.uid() = user_id);

-- ============================================================
-- DEALS policies
-- ============================================================
CREATE POLICY "Users can view their own deals"
  ON deals FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own deals"
  ON deals FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own deals"
  ON deals FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own deals"
  ON deals FOR DELETE
  USING (auth.uid() = user_id);

-- ============================================================
-- ACTIVITIES policies
-- ============================================================
CREATE POLICY "Users can view their own activities"
  ON activities FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own activities"
  ON activities FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own activities"
  ON activities FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own activities"
  ON activities FOR DELETE
  USING (auth.uid() = user_id);
```

---

## 4. Auto-update `updated_at` Trigger

```sql
-- Create a reusable trigger function
CREATE OR REPLACE FUNCTION handle_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables
CREATE TRIGGER set_updated_at_profiles
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION handle_updated_at();

CREATE TRIGGER set_updated_at_companies
  BEFORE UPDATE ON companies
  FOR EACH ROW EXECUTE FUNCTION handle_updated_at();

CREATE TRIGGER set_updated_at_contacts
  BEFORE UPDATE ON contacts
  FOR EACH ROW EXECUTE FUNCTION handle_updated_at();

CREATE TRIGGER set_updated_at_deals
  BEFORE UPDATE ON deals
  FOR EACH ROW EXECUTE FUNCTION handle_updated_at();

CREATE TRIGGER set_updated_at_activities
  BEFORE UPDATE ON activities
  FOR EACH ROW EXECUTE FUNCTION handle_updated_at();
```

---

## 5. Entity Relationship Diagram

```
auth.users (Supabase built-in)
     |
     |-- profiles (1:1)
     |
     |-- companies (1:many)
     |       |
     |       |-- contacts (many:1, optional)
     |       |-- deals    (many:1, optional)
     |       |-- activities (many:1, optional)
     |
     |-- contacts (1:many)
     |       |
     |       |-- deals      (many:1, optional)
     |       |-- activities (many:1, optional)
     |
     |-- deals (1:many)
     |       |
     |       |-- activities (many:1, optional)
     |
     |-- activities (1:many)


Table relationships summary:
  profiles   → auth.users   (1:1, CASCADE DELETE)
  companies  → auth.users   (many:1, CASCADE DELETE)
  contacts   → auth.users   (many:1, CASCADE DELETE)
  contacts   → companies    (many:1, SET NULL on delete)
  deals      → auth.users   (many:1, CASCADE DELETE)
  deals      → contacts     (many:1, SET NULL on delete)
  deals      → companies    (many:1, SET NULL on delete)
  activities → auth.users   (many:1, CASCADE DELETE)
  activities → contacts     (many:1, CASCADE DELETE)
  activities → deals        (many:1, CASCADE DELETE)
  activities → companies    (many:1, CASCADE DELETE)
```

---

## 6. Supabase JS Query Reference

### Fetch all contacts with their company

```typescript
const { data, error } = await supabase
  .from('contacts')
  .select(`
    *,
    company:companies(id, name, domain)
  `)
  .order('created_at', { ascending: false });
```

### Fetch a single contact with company, deals, and activities

```typescript
const { data, error } = await supabase
  .from('contacts')
  .select(`
    *,
    company:companies(id, name, domain, website),
    deals(id, title, value, stage, close_date),
    activities(id, type, subject, body, due_date, completed, created_at)
  `)
  .eq('id', contactId)
  .single();
```

### Fetch all deals with contact and company, ordered by stage

```typescript
const { data, error } = await supabase
  .from('deals')
  .select(`
    *,
    contact:contacts(id, first_name, last_name, email),
    company:companies(id, name)
  `)
  .order('stage')
  .order('created_at', { ascending: false });
```

### Fetch all companies with contact count

```typescript
const { data, error } = await supabase
  .from('companies')
  .select(`
    *,
    contacts(count)
  `)
  .order('name');
```

### Fetch activities for a specific deal

```typescript
const { data, error } = await supabase
  .from('activities')
  .select(`
    *,
    contact:contacts(id, first_name, last_name)
  `)
  .eq('deal_id', dealId)
  .order('created_at', { ascending: false });
```

### Dashboard stats query — total deal value by stage

```typescript
const { data, error } = await supabase
  .from('deals')
  .select('stage, value')
  .eq('user_id', userId);

// Then reduce client-side:
const statsByStage = data.reduce((acc, deal) => {
  acc[deal.stage] = (acc[deal.stage] || 0) + deal.value;
  return acc;
}, {});
```

### Search contacts by name or email

```typescript
const { data, error } = await supabase
  .from('contacts')
  .select('*, company:companies(name)')
  .or(`first_name.ilike.%${query}%,last_name.ilike.%${query}%,email.ilike.%${query}%`)
  .order('first_name');
```

### Paginated contacts

```typescript
const PAGE_SIZE = 20;

const { data, error, count } = await supabase
  .from('contacts')
  .select('*, company:companies(name)', { count: 'exact' })
  .range(page * PAGE_SIZE, (page + 1) * PAGE_SIZE - 1)
  .order('created_at', { ascending: false });
```
