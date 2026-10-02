# Rappi deal monitor (Pereira, Colombia)

Python script run by GitHub Actions every 30 minutes (`.github/workflows/monitor.yml`). It scans rappi.com.co near the user's address and pushes deals of 60%+ to ntfy. Claude is not used at runtime; only to maintain it when Rappi changes its site.

## Layout

- `monitor/main.py`: entry point (`python -m monitor`). Runs the sections, filters already-notified deals, sends notifications.
- `monitor/scan.py`: restaurants (browser), stores (HTTP, 40 per round with a saved cursor), chains (backup).
  - Store lists come from `/<CIUDAD>/tiendas/tipo/<type>`; Colombian store IDs are per city, so the city prefix is required.
- `monitor/browser.py`: Playwright. Opens `/restaurantes` with the `currentLocation` cookie, clicks `#popular_filters-Promos`, captures `restaurants-bus/stores/filters` and replays it with `filters.discounts.types` = `["offer_by_product"]` / `["percentage"]`, then reads `restaurants-bus/store/id/<id>/` per candidate (`is_prime` = `RAPPI_PRO`).
- `monitor/parsers.py`: pure functions over `__NEXT_DATA__` and Rappi JSON. `discount_pct` truncates (matches what Rappi CO displays).
- `monitor/notify.py`: ntfy formatting (COP prices `$12.900`) and sending.
- `monitor/state.py`: `state/state.json`, committed by the workflow. Keys are SHA-256 hashes.
- `monitor/config.py`: environment variables (table in README.md).
- `tests/`: pytest with trimmed real Rappi pages (Peru originals; parsing is the same). `tests/test_browser.py` runs the whole monitor against a local fake Rappi.

## Commands

```bash
.venv/Scripts/python -m pytest -q
RAPPI_UBICACION="4.81, -75.69" RAPPI_PRO=si .venv/Scripts/python -m monitor --sin-enviar --prueba
```

## Rules

- Never commit the user's address/coordinates or ntfy topic; they live only in the `RAPPI_UBICACION` and `NTFY_TOPIC` repository secrets.
- The repo lives on the user's personal GitHub account (Sebaa711), not the work account.
- Do not evade Rappi rate limits: a 403/429 stops the round on purpose.
