# HYPER AI CRYPTO IQ BOT — Free private cloud saving (Android)

**Current status:** The website includes optional Supabase cloud sync, but it is NOT active until you connect your own Supabase project and sign in. No real exchange orders or crypto wallet connection.

## 1. Create a free Supabase project

1. In Chrome on Android, open https://supabase.com/dashboard and sign up or sign in.
2. Choose **New project**, give it a name such as `hyper-iq-paper`, choose a region and a strong database password. Keep the database password private.
3. Wait for the project to finish provisioning.
4. Open **SQL Editor** → **New query**. Paste and run the SQL below once.

## 2. Secure database table and per-user permissions

```sql
create table if not exists public.hyper_iq_paper_accounts (
  user_id uuid primary key references auth.users (id) on delete cascade,
  state jsonb not null check (jsonb_typeof(state) = 'object'),
  updated_at timestamptz not null default now()
);

alter table public.hyper_iq_paper_accounts enable row level security;

-- Clients without a signed-in user must not read or modify paper accounts.
revoke all on table public.hyper_iq_paper_accounts from public;
revoke all on table public.hyper_iq_paper_accounts from anon;
grant usage on schema public to authenticated;
grant select, insert, update on table public.hyper_iq_paper_accounts to authenticated;

drop policy if exists "hyper_iq_account_select_own" on public.hyper_iq_paper_accounts;
drop policy if exists "hyper_iq_account_insert_own" on public.hyper_iq_paper_accounts;
drop policy if exists "hyper_iq_account_update_own" on public.hyper_iq_paper_accounts;

create policy "hyper_iq_account_select_own"
on public.hyper_iq_paper_accounts
for select to authenticated
using (user_id = (select auth.uid()));

create policy "hyper_iq_account_insert_own"
on public.hyper_iq_paper_accounts
for insert to authenticated
with check (user_id = (select auth.uid()));

create policy "hyper_iq_account_update_own"
on public.hyper_iq_paper_accounts
for update to authenticated
using (user_id = (select auth.uid()))
with check (user_id = (select auth.uid()));
```

**Security:** Do not remove RLS or use an unrestricted policy. This app must only use the Supabase **publishable** key (`sb_publishable_...`; legacy `anon` key also works). Never paste the Supabase database password, service-role key, secret key (`sb_secret_...`), wallet private key, or seed phrase into the bot or GitHub source.

## 3. Configure email sign-in redirect

In Supabase, open **Authentication → URL Configuration**. Set **Site URL** and add a **Redirect URL** for:

`https://jzonga.github.io/hyper-ai-crypto-iq-bot-/`

Make sure the Email authentication provider is enabled. A magic-link email must open this exact GitHub Pages site.

## 4. Connect your Android HYPER IQ app

1. In Supabase **Project Settings → API Keys** (or **Connect**), find **Project URL** and the **publishable key**. These two are intended for browser use with RLS; never use a secret key.
2. Open https://jzonga.github.io/hyper-ai-crypto-iq-bot-/ and find **Settings → Secure cloud paper account**.
3. Enter the URL and publishable key. Tap **Save project & connect**.
4. Enter your email, tap **Email me sign-in link**, and open the link from your email.
5. Back in the app, go to **Settings** and confirm you're signed in. If there is no existing cloud account, tap **Use this account · enable automatic saving**. Confirm.
6. The app now sends account updates when trades/settings/history change, while you're online and signed in.

## 5. Using another phone/browser

Sign into the same Supabase project and email account; tap **Restore cloud account**. When cloud and browser records differ, choose which copy to use. The app does not silently overwrite a nonempty browser record. **AUTO always restarts OFF**.

## Important limits

- If the site says cloud disconnected, **only local paper data is being saved**. Download a JSON backup as well.
- The two previously missing MINTER/Chaton transactions cannot be recovered just by enabling a database now.
- Free-tier projects have usage limits and may pause after inactivity; check Supabase's current pricing.
- Supabase stores simulated balances, positions and history only. It does not run your trading bot in the background. AUTO paper trading still requires an active website and fresh quotes.
- Chart prices, screening scores, RSI, EMA, and security reports are not predictions or guarantees of profit.
