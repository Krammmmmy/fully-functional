# PetWatch dashboard integration notes

## Environment

Copy `.env.example` to `.env.local` and enter your Supabase project URL and publishable key. Do not put a Supabase service-role key in a browser application.

## Tables used by the dashboard

- `pet_schedules`: schedule create, read, update, and delete.
- `sensor_readings`: latest food/water weight, ultrasonic level, temperature, and humidity.
- `device_status`: device online flag and last-seen time.
- `device_commands`: manual dispense command queue.
- `feeding_history`: completed/failed dispensing activity.
- `alerts`: device alerts and read state.

The dashboard polls sensor readings, device status, history, and alerts every five seconds. The browser's publishable key must have appropriate Supabase Row Level Security policies for the required operations. Keep RLS enabled and grant only the operations needed by the intended user roles. Do not expose a service-role key in the frontend.

## Important firmware mismatch

The supplied firmware polls `feeding_schedules`, but the dashboard writes `pet_schedules`. Until firmware is changed, schedule entries will save and display in the dashboard but will not run automatically. The firmware also currently updates `device_status.is_online` without refreshing `last_seen`, so the dashboard cannot use `last_seen` as a reliable heartbeat until firmware updates it on each successful connection/status check.

## Data interpretation

The HX711 reports weight in grams. The dashboard therefore displays both food and water dispensing quantities in grams; it does not relabel water weight as millilitres. Food/water level percentages come from the ultrasonic readings.

## Test sequence

1. Set `.env.local` values and start the dashboard.
2. Confirm the latest row in `sensor_readings` changes when the ESP32 uploads data.
3. Confirm `device_status` contains a PetWatch row and `is_online` is updated.
4. Press the manual fill button and confirm two `pending` rows appear in `device_commands`.
5. Confirm the ESP32 changes both rows to `processing`, then `completed` or `failed`.
6. Confirm the ESP32 adds actual results to `feeding_history`.
7. Create/edit/delete a schedule and verify it in `pet_schedules`. Do not rely on automatic execution until the firmware table mismatch is fixed.
