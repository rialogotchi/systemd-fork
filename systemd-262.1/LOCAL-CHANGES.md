# Local changes

## 1. Everything off by default, RAM only, upload only to the configured URL
- journald default storage is `volatile` (/run/log/journal, RAM). Build option
  `-Djournal-storage-default=volatile` is the new default.
- Metrics/reports are only generated on demand and live in RAM (stdout / process memory);
  nothing is written to disk.
- Nothing is sent anywhere by default. `systemd-journal-upload` / `journal-remote` stay disabled
  in the presets. `systemd-report upload` refuses to run while no URL is configured.
- The `systemd-report*.socket` units (local AF_UNIX metric sources only) stay enabled so that
  `systemd-report generate/upload` has data; they never talk to the network.
- NEW: /etc/systemd/report.conf (+ report.conf.d/*.conf), section [Report]:
      URL=https://your.server/ingest
      Header=Authorization: Bearer TOKEN
      NetworkTimeoutSec=30s
      ServerKeyFile= / ServerCertificateFile= / TrustedCertificateFile=
  `systemd-report upload` sends ONLY to that URL. The --url/--key/--cert/--trust/
  --extra-header/--network-timeout switches were removed, the "upload via
  /run/systemd/report.upload/ sockets" path was removed, and the insecure "trust all" TLS mode
  was removed. URL must be a valid http(s) URL.
- Logs to your URL (separate, optional): build with `-Dremote=enabled -Dlibcurl=enabled`,
  set URL= in /etc/systemd/journal-upload.conf.d/, then
  `systemctl enable --now systemd-journal-upload.service`.

## 2. birthDate (age data) removed
Removed `birthDate` from user records, `homectl --birth-date=`, `homectl inspect` output,
parse_birth_date()/BIRTH_DATE_* helpers, tests, man/homectl.xml, docs/USER_RECORD.md, NEWS.
parse_calendar_date() restored to the simple form.
