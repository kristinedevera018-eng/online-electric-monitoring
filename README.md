# Smart-Grid Intelligence Platform

Streamlit dashboard for household accounts, meter readings, billing,
analytics, incident reporting, and GIS views.

## Run on Windows

The app and its database use paths relative to this project folder, so it can
be extracted anywhere. From PowerShell, change to the extracted
`smart-grid-monitor` folder and install its dependencies:

```powershell
cd "C:\path\to\smart-grid-monitor"
python -m pip install -r requirements.txt
```

Copy `.streamlit\secrets.toml.example` to `.streamlit\secrets.toml` and replace
the placeholder values with your own administrator credentials. The app can
start without this file, but administrator sign-in remains disabled until it
is configured. Do not commit `secrets.toml`.

Start the app from PowerShell:

```powershell
python -m streamlit run app.py
```

Alternatively, double-click `run.bat`; it starts Streamlit from the folder
where the project was extracted. To access the app from another device on the
same network, use:

```powershell
python -m streamlit run app.py --server.address 0.0.0.0
```

Then open `http://<computer-ip>:8501` on that device. The computer's firewall
must allow connections to port 8501. For access outside the local network,
deploy the app to a hosting service rather than exposing the development
server directly.

The project folder must be writable because the app stores its SQLite
database and generated reports under `data\`.

## Deploy online with Streamlit Community Cloud

1. Push this project folder to a GitHub repository. Do not upload
   `.streamlit/secrets.toml`, the local database, or generated reports; these
   are excluded by `.gitignore`.
2. In Streamlit Community Cloud, create an app from that repository and select
   the branch containing the app.
3. Set **Main file path** to `app.py` if this folder is the repository root,
   or `smart-grid-monitor/app.py` if the repository contains the parent folder.
4. In the app's **Settings → Secrets**, add TOML with a unique, strong
   administrator credential:

   ```toml
   [admin]
   username = "your-admin-username"
   password = "use-a-long-unique-password"
   ```

5. Deploy, then test resident registration and administrator sign-in on the
   hosted URL.

The app currently stores records in a local SQLite file at `data/`. That file
is suitable for local testing, but hosted app files may be replaced and should
not be treated as durable storage. Do not use this deployment for real
household or payment data until the database is moved to a managed, persistent
database and data handling has been reviewed. The payment screens record
payments in the app; they do not by themselves charge a real payment provider.

To use managed PostgreSQL (for example, Neon), add the provider's SSL-enabled
connection URL to Streamlit Cloud Secrets:

```toml
[database]
url = "postgresql://user:password@host/database?sslmode=require"
```

When this setting is present, the app creates its schema and demo baseline in
that PostgreSQL database on first startup. Without it, the app uses local
SQLite. The local SQLite database is never copied to PostgreSQL automatically.
Keep the connection URL private, use a fresh database for a demo deployment,
and verify the app's access restrictions in Streamlit Cloud before sharing it.

## Demo resident accounts

In the currently provisioned local database, seeded accounts `ACC-8001`
through `ACC-8010` use the shared temporary password provided to the operator.
New resident registrations must use an unused account number; existing
accounts cannot be claimed through sign-up. The temporary password is not
stored in source control.

The SQLite database is stored at `data\electricity_monitor.db` and is
initialized automatically when the app starts. Do not run
`python database.py --reset` unless you intend to erase and reseed that data.

Household map pins use the latitude and longitude stored on each household.
Seeded demo coordinates are approximate barangay or locality positions, not
exact household locations. Echague demo pins now use mapped locations in
Echague rather than the old points north of its town area. An administrator
can replace any approximate pin with verified GPS or map-pin coordinates from
the Households page. Households without saved coordinates are estimated from
mapped households in the same barangay when possible. Emergency reports
contain a barangay rather than an exact location, so their map pins are
approximate barangay locations.

New resident registrations and administrator-added households must provide
both latitude and longitude. The database validates coordinate ranges and
enforces account-number uniqueness without regard to letter case. During
initialization, the database adds 19 synthetic demo households when a seeded
demo password is available. In the current provisioned database, this brings
the household total to 30. These accounts reuse the configured demo-account
password hash; the password itself
is not stored in source control. Their location pins use mapped community or
public-place coordinates from OpenStreetMap and are locality approximations,
not exact home addresses. Synthetic residential accounts use person names and
commercial accounts use business names; all remain synthetic demo records.

Synthetic households have varied account statuses, meter usage, and payment
histories. Payment references prefixed with `SYN-` identify generated demo
transactions; unpaid readings remain visible as outstanding statements. This
additive migration does not replace existing households or payments.

Regenerate the Markdown executive summary with:

```powershell
python executive_report.py
```