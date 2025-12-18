# First Touch Rate Discrepancy Helper

This repo contains the `fill_form.py` tool we use to gather shipment/package details and rates from ShipperHQ logs to complete the First Touch Rate Discrepancy form.

## Quick start (support agents)

1) Pick or create a folder to hold the tool (anywhere is fine), then clone the repo:
   ```bash
   git clone https://github.com/FernandoSHQ/Fill-Forms.git
   cd /path/to/Fill-Forms > Navigate into the folder you created
   ```
2) Install deps if needed:
   ```bash
   pip install -r requirements.txt
   ```
3) Navigate back into the ops-tools directory. Link `findlog.sh` from the ops-tools repo (so `fill_form.py` can auto-fetch logs by transaction ID). From inside your ops-tools directory run:
   ```bash
   ln -s "$(pwd)/findlog.sh" /path/to/Fill-Forms/findlog.sh
   chmod +x "$(pwd)/findlog.sh"
   ```
   Replace `/path/to/Fill-Forms` with where you cloned the repo in step 1.
4) Add this folder to your PATH so you can run `fill_form.py` from anywhere:
   ```bash
   echo 'export PATH="/path/to/Fill-Forms:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

## Running the tool

With a log file you already have:
```bash
fill_form.py shipperws.log
```

If you run it with no arguments, it will pop a file picker so you can select a log.

Output: a `rate_analysis_<timestamp>.txt` is written next to the log file you processed. This text file has the ship-from/ship-to, cart items, returned carrier services, and the per-box packing list you need for the First Touch Rate Discrepancy form.

To update the tool later (grab latest fixes):
```bash
cd /path/to/Fill-Forms
git pull
```

## Notes

- Use ShipperWS logs when possible (they include both request and response JSON).
- If you see a parsing error, double-check you selected the right log file and that it contains the request/response blocks.
