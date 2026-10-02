# Rappi deal monitor (Pereira)

Sends a push notification to your phone (via the free **ntfy** app) when Rappi Colombia has
discounts of **60% or more** near your address: restaurants and stores (supermarkets,
pharmacies, liquor, express, Rappi Mall). It runs on GitHub Actions every 30 minutes, needs
no PC, and never repeats a deal you were already notified about (for 7 days).

Adapted from [aaronortecho-tech/rappi-ofertas](https://github.com/aaronortecho-tech/rappi-ofertas)
(Rappi Peru), trimmed to the Rappi monitor and converted to rappi.com.co.

## What it checks

| Section | How | Each round |
|---|---|---|
| Restaurants | Headless Chromium opens rappi.com.co with your location, turns on the **Promos** filter and reads the live menu of every open restaurant announcing the minimum discount. Rappi Pro deals included (`RAPPI_PRO`). | All of them |
| Stores | Reads the **Ofertas** page of every store Rappi lists for your city (`/pereira/tiendas/tipo/...`). | 40 stores, rotating (~112 in Pereira, full cycle every ~1.5 h) |
| Chains (backup) | KFC, El Corral, Frisby, Domino's, Kokoriko, Subway, Qbano, Crepes & Waffles, McDonald's. | Only if the restaurant list fails |

Notifications:

- One notification per store with its best deals; tapping it opens the store in Rappi.
- Deals of **80% or more** arrive as max-priority alarms.
- More than 8 stores in one round → the rest arrive in a single summary.
- If Rappi changes its site and 3 rounds in a row fail, you get at most one warning per day.

## Setup

1. Install **ntfy** on your phone ([Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy) / [iPhone](https://apps.apple.com/app/ntfy/id1625396347)) and subscribe to your topic (the name stored in the `NTFY_TOPIC` secret). Anyone who knows the topic name can read it, so keep it long and random.
2. Repository secrets (Settings → Secrets and variables → Actions → Secrets):
   - `NTFY_TOPIC`: your ntfy topic.
   - `RAPPI_UBICACION`: `latitude, longitude` of your address (e.g. from Google Maps).
3. Run it once by hand: Actions → *Monitor de ofertas Rappi* → Run workflow (sends a test notification).

## Optional settings (Settings → Secrets and variables → Actions → **Variables**)

| Variable | Default | Meaning |
|---|---|---|
| `DESCUENTO_MINIMO` | `60` | Minimum discount (%) to notify |
| `ALARMA_DESDE` | `80` | From this % the notification is an alarm |
| `HORAS_SILENCIO` | none | e.g. `23-7`: notifications in that window arrive silently (Colombia time) |
| `RAPPI_PRO` | `si` | Include Rappi Pro-only deals |
| `CIUDAD` | `pereira` | City slug as Rappi uses it in URLs |
| `DISTANCIA_MAXIMA_KM` | none | Ignore restaurants farther than this |
| `TIPOS_TIENDA` | `market, super-farma, express-big, vinos-y-licores, rappimall-parent` | Store types to scan |
| `TIENDAS_POR_RONDA` | `40` | Stores per round |
| `REPETIR_AVISO_HORAS` | `168` | Hours before the same deal can be notified again |
| `MAX_AVISOS_POR_RONDA` | `8` | Individual notifications per round before summarizing |
| `REVISAR_TIENDAS` / `REVISAR_RESTAURANTES` | `si` | Turn a section off |
| `REVISAR_RAPPI_MARKET` | `no` | Rappi Turbo/Market (not available in Pereira) |
| `DETALLE_EN_LOGS` | `no` | Verbose logs |

## Running locally

```bash
python -m venv .venv
.venv/Scripts/python -m pip install -r requirements-dev.txt   # Windows path; use .venv/bin on Linux/macOS
.venv/Scripts/python -m playwright install chromium
.venv/Scripts/python -m pytest -q                              # all tests must pass
RAPPI_UBICACION="4.81, -75.69" .venv/Scripts/python -m monitor --sin-enviar --prueba   # real scan, prints instead of sending
```

## Notes

- Percentages are truncated like Rappi Colombia displays them ($180.000 before $449.900 is 59%, not 60%).
- Discounts are what Rappi publishes; it does not verify that the "before" price was ever real.
- The memory (`state/state.json`) only stores SHA-256 hashes of notified deals and the store rotation cursor.
