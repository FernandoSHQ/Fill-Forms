# First Touch Rate Discrepancy Helper

This repo contains the `fill_form.py` tool we use to gather shipment/package details and rates from ShipperHQ logs to complete the First Touch Rate Discrepancy form.

## Quick start

```bash
git clone https://github.com/FernandoSHQ/Fill-Forms.git
cd Fill-Forms
pip install -r requirements.txt   # if you need any deps; skip if already set up
```

## Running the tool

With a log file you already have:
```bash
python3 fill_form.py /path/to/shipperws_or_carrierws.log
```

If you run it with no arguments, it will pop a file picker so you can select a log.

Output: a `rate_analysis_<timestamp>.txt` is written next to the log file you processed. This text file has the ship-from/ship-to, cart items, returned carrier services, and the per-box packing list you need for the First Touch Rate Discrepancy form.

## Notes

- Use ShipperWS logs when possible (they include both request and response JSON).
- If you see a parsing error, double-check you selected the right log file and that it contains the request/response blocks.
