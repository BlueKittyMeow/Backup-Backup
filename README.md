# Backup-Backup
The backup to the backup


# Family Archive & Backup Plan — FINAL
### January 2026

---

## The Situation

Thanks to the AI boom sucking up all global RAM production, used servers that were $300-450 a year ago are now $1,200+, and even a basic Synology NAS jumped to $900. So we're going with the smart play: my beefy gaming/AI rig does double duty as the storage server.

Thanks, Sam. 🙄

---

## What I'm Building

```
┌─────────────────────────────────────────────────────────────────┐
│                MY GAMING/AI PC (lives in my room)               │
│                                                                 │
│   RTX 4070 Ti │ 32GB RAM │ Running TrueNAS VM or ZFS-on-Linux   │
│                                                                 │
│         ┌─────────────────────────────────────────┐             │
│         │            4-BAY DAS (USB 3.2)          │             │
│         │                                         │             │
│         │   4 × 8TB in RAID-Z2 = 16TB usable      │             │
│         │   ZFS checksums + self-healing          │             │
│         │                                         │             │
│         │   Mom's data: ~750 GB                   │             │
│         │   My data: ~3 TB                        │             │
│         │   VHS/photo digitization: ~1.5 TB       │             │
│         │   Room to grow: ~10 TB                  │             │
│         └─────────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
         │
         │ Nightly (restic, runs while I sleep)
         ▼
┌─────────────────────────────────────────────────────────────────┐
│              AWS GLACIER DEEP ARCHIVE — ~$4-8/month             │
│                                                                 │
│   Encrypted, versioned, offsite                                 │
│   Protects against: fire, flood, ransomware, my mistakes        │
└─────────────────────────────────────────────────────────────────┘

         +

┌─────────────────────────────────────────────────────────────────┐
│                    M-DISC PHYSICAL ARCHIVE                      │
│                                                                 │
│   ~500 GB of truly irreplaceable stuff                          │
│   Burned to 100GB BDXL M-Discs                                  │
│   Stored at boyfriend's place                                   │
│   Rated for 100+ years (NIST verified)                          │
│   Protects against: EMP, total societal collapse, zombies       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Data Inventory

### Mom's Data — COMPLETE ✓

| Source | Size |
|--------|------|
| iCloud | 626.4 GB |
| katieogill@gmail.com | 100 GB |
| calicat611@gmail.com | 0 |
| debbiewilliamsgoss@gmail.com | 0 |
| kyotecollie@gmail.com | 0 |
| laylacolliedog@gmail.com | 0 |
| texasyellowrosebud@gmail.com | 0.11 GB |
| yellowroseoftexas.texas@gmail.com | 0.03 GB |
| 2019 MacBook (local) | ~50 GB est |
| **Mom's Total** | **~780 GB** |

*Note: 6 of 7 Google accounts were collie-themed tumbleweeds*

### Your Data

| Source | Size |
|--------|------|
| Google Drive | ~2,000 GB |
| Local hard drives | ~1,000 GB |
| "Hidden chaos gremlins" | ??? |
| **My Total** | **~3,000 GB+** |

### Future Digitization Projects

| Source | Estimated Size |
|--------|----------------|
| VHS tapes (lossless masters) | ~1,000-1,500 GB |
| Photo/negative scans | ~100-200 GB |
| **Digitization Total** | **~1,200-1,700 GB** |

*Partner has full VHS digitization setup (TBC, VCR, capture card) — road trip with boxes of tapes to his place*

### Combined Storage Need

| Category | Size |
|----------|------|
| Mom's stuff | ~780 GB |
| Your stuff | ~3,000 GB |
| Digitization projects | ~1,500 GB |
| **Raw total** | **~5,300 GB** |
| Post-dedup estimate | ~4,000 GB |
| Growth headroom | ~12,000 GB |
| **Target** | **16 TB usable** |

---

## Shopping List

### Storage Hardware

| Item | Price | Notes |
|------|-------|-------|
| 4-bay DAS enclosure | ~$120-150 | TerraMaster D4-300, ORICO, Yottamaster |
| 4 × 8TB WD Red Plus (WD80EFPX) | ~$600 | Or Seagate IronWolf |
| **Storage subtotal** | **~$720-750** | |

### M-Disc Archive

| Item | Price | Notes |
|------|-------|-------|
| Verbatim M-DISC BDXL 100GB 10-pack | ~$125-140 | Enough for ~1TB of critical data |
| BDXL burner | $0 | I already have one! |
| **M-Disc subtotal** | **~$125-140** | |

### Power Protection

| Item | Price | Notes |
|------|-------|-------|
| UPS (battery backup) | ~$100-180 | APC Back-UPS 900VA (~$100) or CyberPower CP1500AVRLCD (~$150) |
| **Power subtotal** | **~$100-180** | |

**Why I need a UPS:** ZFS really doesn't like sudden power loss mid-write. A UPS gives the system time to flush writes and shut down cleanly during an outage. Also protects against surges.

---

## Grand Total

| Category | Cost |
|----------|------|
| DAS + drives | $720-750 |
| M-Discs | $125-140 |
| UPS | $100-180 |
| **One-time total** | **$945-1,070** |
| Glacier (annual) | ~$48-96/year |

**Money saved vs. alternatives:**
- vs. Synology DS923+ ($900 + $600 drives): **~$550 saved**
- vs. Used R720 ($1,200+ + $600 drives): **~$850+ saved**

---

## Folder Structure

```
/mnt/vault/
├── mom/
│   ├── icloud/
│   │   └── Photos/
│   ├── google-photos/
│   │   └── katieogill/
│   └── laptop-backup/
│
├── you/
│   ├── google-drive/
│   ├── local-archive/
│   ├── photos/
│   │   ├── family-safe/
│   │   └── personal/        ← spicy memes quarantine zone
│   └── projects/
│
├── digitization/
│   ├── vhs-masters/         ← lossless captures
│   ├── vhs-compressed/      ← viewing copies
│   ├── photo-scans/
│   └── negatives/
│
├── shared/
│   └── family-photos/       ← deduplicated best-of for sharing
│
└── system/
    ├── manifests/           ← M-Disc contents, Glacier inventory
    └── scripts/             ← backup automation
```

---

## Daily Sync Automation

This is how new photos and files automatically flow from cloud services to my NAS every day.

### The Flow

```
┌──────────────────┐     2 AM       ┌─────────────────────┐
│  Mom's iCloud    │ ──────────────►│                     │
│  (626 GB)        │   icloudpd     │                     │
└──────────────────┘                │                     │
                                    │     MY NAS          │
┌──────────────────┐     3 AM       │   /mnt/vault/       │
│  Mom's Google    │ ──────────────►│                     │
│  Photos (100 GB) │  gphotos-sync  │                     │
└──────────────────┘                │                     │
                                    │                     │
┌──────────────────┐     4 AM       │                     │
│  My Google       │ ──────────────►│                     │
│  Drive (2 TB)    │   rclone       │                     │
└──────────────────┘                └──────────┬──────────┘
                                               │
                                               │ 5 AM
                                               │ restic
                                               ▼
                                    ┌─────────────────────┐
                                    │   AWS Glacier       │
                                    │   (encrypted)       │
                                    └─────────────────────┘
```

---

### Tool 1: rclone (My Google Drive)

**What it does:** Syncs Google Drive (and 40+ other cloud services) to local folders. Gold standard.

**Install:**
```bash
# Ubuntu/Debian
sudo apt install rclone

# Or get latest
curl https://rclone.org/install.sh | sudo bash
```

**One-time setup:**
```bash
rclone config
# Choose: n (new remote)
# Name: gdrive
# Type: drive (Google Drive)
# Follow OAuth prompts (opens browser)
```

**Test it:**
```bash
# Dry run first (shows what would happen)
rclone sync gdrive: /mnt/vault/you/google-drive/ --dry-run

# Actually sync
rclone sync gdrive: /mnt/vault/you/google-drive/ --progress
```

**Useful flags:**
```bash
--bwlimit 10M          # Limit bandwidth to 10 MB/s
--transfers 4          # Parallel transfers
--log-file=/var/log/rclone.log
```

---

### Tool 2: icloudpd (Mom's iCloud Photos)

**What it does:** Downloads all photos from iCloud. Runs incrementally (only grabs new stuff).

**Install:**
```bash
pip install icloudpd
```

**One-time setup (need mom's phone for 2FA):**
```bash
icloudpd --directory /mnt/vault/mom/icloud/ \
         --username moms-apple-id@email.com \
         --size original
# Enter password when prompted
# Enter 2FA code from mom's phone
```

**After first run:** It stores a session cookie, so future runs don't need 2FA (until the session expires, usually a few months).

**Useful flags:**
```bash
--auto-delete          # Remove local files deleted from iCloud
--until-found 50       # Stop after finding 50 already-downloaded photos (faster)
--recent 100           # Only check last 100 photos (fastest for daily runs)
```

**⚠️ Gotcha:** When the session expires, it fails silently. Check logs periodically or set up alerts.

---

### Tool 3: gphotos-sync (Mom's Google Photos)

**What it does:** Syncs Google Photos to local folder. Google made this harder than necessary, but it works.

**Install:**
```bash
pip install gphotos-sync
```

**One-time setup:**
```bash
gphotos-sync /mnt/vault/mom/google-photos/katieogill/
# Opens browser for Google OAuth
# Authorize the app
```

**After first run:** Stores credentials locally, future runs are automatic.

**Alternative: Google Takeout (manual)**

If automated sync is too finicky, just do quarterly exports:
1. Go to takeout.google.com (logged into mom's account)
2. Select Google Photos only
3. Choose .zip, 10GB chunks
4. Download all chunks
5. Extract to NAS
6. Run dedup (czkawka) to avoid duplicates

---

### Putting It Together: The Cron Schedule

Create a script at `/mnt/vault/system/scripts/daily-sync.sh`:

```bash
#!/bin/bash
# Daily cloud sync script
# Runs overnight, syncs all cloud sources, then backs up to Glacier

LOG=/var/log/daily-sync.log
VAULT=/mnt/vault

echo "==============================" >> $LOG
echo "Sync started: $(date)" >> $LOG
echo "==============================" >> $LOG

# 1. Sync my Google Drive
echo "[$(date +%H:%M)] Starting Google Drive sync..." >> $LOG
rclone sync gdrive: $VAULT/you/google-drive/ \
    --log-file=$LOG \
    --log-level INFO \
    --bwlimit 10M

# 2. Sync mom's iCloud
echo "[$(date +%H:%M)] Starting iCloud sync..." >> $LOG
icloudpd --directory $VAULT/mom/icloud/ \
    --username moms-apple-id@email.com \
    --size original \
    --until-found 50 \
    >> $LOG 2>&1

# 3. Sync mom's Google Photos
echo "[$(date +%H:%M)] Starting Google Photos sync..." >> $LOG
gphotos-sync $VAULT/mom/google-photos/katieogill/ >> $LOG 2>&1

# 4. Backup everything to Glacier
echo "[$(date +%H:%M)] Starting Glacier backup..." >> $LOG
export AWS_ACCESS_KEY_ID=your-key-here
export AWS_SECRET_ACCESS_KEY=your-secret-here
export RESTIC_PASSWORD=your-restic-password

restic -r s3:s3.amazonaws.com/your-bucket-name \
    backup $VAULT/mom $VAULT/you \
    --exclude-caches \
    >> $LOG 2>&1

# 5. Prune old Glacier snapshots
restic -r s3:s3.amazonaws.com/your-bucket-name \
    forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune \
    >> $LOG 2>&1

echo "[$(date +%H:%M)] Sync complete!" >> $LOG
```

Make it executable:
```bash
chmod +x /mnt/vault/system/scripts/daily-sync.sh
```

**Schedule with cron:**
```bash
crontab -e

# Add this line (runs at 2 AM daily):
0 2 * * * /mnt/vault/system/scripts/daily-sync.sh
```

---

### Monitoring: Know When Things Break

**Option 1: Check logs manually**
```bash
tail -100 /var/log/daily-sync.log
```

**Option 2: Email on failure**

Add to end of script:
```bash
if [ $? -ne 0 ]; then
    echo "Backup failed on $(date)" | mail -s "⚠️ BACKUP FAILED" you@email.com
fi
```

**Option 3: Healthchecks.io (free, recommended)**

1. Sign up at healthchecks.io
2. Create a check, get my unique URL
3. Add to end of script:
```bash
curl -fsS --retry 3 https://hc-ping.com/your-unique-id
```

If the ping doesn't arrive on schedule, I get an email/push alert.

---

### Quick Reference: Sync Commands

**Check what rclone would do:**
```bash
rclone sync gdrive: /mnt/vault/you/google-drive/ --dry-run
```

**Force iCloud re-auth (when session expires):**
```bash
icloudpd --directory /mnt/vault/mom/icloud/ \
         --username moms-apple-id@email.com \
         --password  # Will prompt for password + 2FA
```

**Check last sync status:**
```bash
tail -50 /var/log/daily-sync.log
```

**Manual sync right now:**
```bash
/mnt/vault/system/scripts/daily-sync.sh
```

---

## Implementation Phases

### Phase 1: Hardware Setup (Weekend Project)

- [ ] Order DAS enclosure
- [ ] Order 4 × 8TB drives
- [ ] Order UPS (APC 900VA ~$100 or CyberPower 1500VA ~$150)
- [ ] Decide: TrueNAS VM vs. ZFS-on-Linux vs. dual boot
- [ ] Set up ZFS pool (RAID-Z2)
- [ ] Configure snapshot schedule (hourly/daily/weekly)
- [ ] Connect UPS, configure auto-shutdown on power loss
- [ ] Test write speeds, verify pool health

### Phase 2: Data Migration (1-2 Weeks)

- [ ] Create folder structure
- [ ] Download mom's iCloud (626 GB)
  - Use iCloud for Windows or `icloud-photos-downloader` CLI
- [ ] Download mom's Google Photos (100 GB)
  - Google Takeout: takeout.google.com
- [ ] Download my Google Drive (2 TB)
  - Google Takeout or rclone
- [ ] Consolidate my local hard drives
- [ ] Run deduplication pass (czkawka)
- [ ] Verify all data present and accessible

### Phase 3: Cloud Backup Setup (Few Hours)

- [ ] Create AWS account (if needed)
- [ ] Create S3 bucket for backups
- [ ] Set up lifecycle rule: move to Glacier Deep Archive after 1 day
- [ ] Install restic on my PC
- [ ] Configure restic repo pointing to S3
- [ ] Run first full backup (will take a while!)
- [ ] Set up nightly cron job / scheduled task
- [ ] Test restore of a few random files
- [ ] Create manifest document

**Restic quick start:**
```bash
# Initialize repo
restic -r s3:s3.amazonaws.com/your-bucket init

# Run backup
restic -r s3:s3.amazonaws.com/your-bucket backup /mnt/vault/mom /mnt/vault/you

# Check snapshots
restic -r s3:s3.amazonaws.com/your-bucket snapshots

# Restore (if ever needed)
restic -r s3:s3.amazonaws.com/your-bucket restore latest --target /mnt/restore
```

### Phase 4: M-Disc Archive (Afternoon Project)

- [ ] Find my BDXL burner
- [ ] Order M-Disc media
- [ ] Identify truly irreplaceable data (~500 GB):
  - Mom's deduplicated photo originals
  - My irreplaceable photos
  - Critical documents (birth certs, tax, legal)
  - NOT: movies, music, memes, Linux ISOs
- [ ] Organize into ~100 GB chunks
- [ ] Burn discs with verification
- [ ] Create printed manifest (disc #, date, contents summary)
- [ ] Send set to boyfriend
- [ ] Store manifest in multiple places (NAS, Google Drive, printed)

### Phase 5: Future — VHS Digitization (Road Trip!)

- [ ] Gather all VHS tapes (parents' wedding, childhood stuff)
- [ ] Drive to partner's place with boxes
- [ ] Use his TBC + VCR + capture setup
- [ ] Capture to lossless format (40-60 GB per 2-hour tape)
- [ ] Create compressed viewing copies
- [ ] Transfer masters to NAS
- [ ] Add to Glacier backup rotation
- [ ] Burn most precious ones to M-Disc

### Phase 6: Future — Jellyfin Media Server

- [ ] Repurpose little Dell square PC (or a Pi)
- [ ] Attach cheap unprotected drives (no RAID needed for media)
- [ ] Install Jellyfin
- [ ] Point at /shared/family-photos and any movies/TV
- [ ] Share with family

---

## Backup Verification Schedule

| Task | Frequency | How |
|------|-----------|-----|
| ZFS scrub | Monthly | `zpool scrub vault` (runs automatically if configured) |
| Check snapshot list | Weekly | `zfs list -t snapshot` |
| Check Glacier backup logs | Weekly | restic snapshots command |
| Test restore from Glacier | Quarterly | Restore a few random files, verify integrity |
| M-Disc spot check | Annually | Read a random disc, verify files open |
| Update M-Disc archive | Annually | Burn new discs for major additions |

---

## Security & "Don't Nuke Anything" Checklist

- [ ] **ZFS snapshots enabled** — hourly, daily, weekly retention
- [ ] **Glacier backups are versioned** — deletions don't propagate
- [ ] **restic encrypts everything** — even AWS can't read your data
- [ ] **Separate user permissions** — mom's folder vs mine
- [ ] **UPS protects against power events** — no mid-write corruption
- [ ] **M-Discs are write-once** — can't accidentally overwrite
- [ ] **Manifest in 3 places** — NAS, cloud, printed

---

## Quick Reference Commands

**ZFS health:**
```bash
zpool status vault
zpool list
```

**Start a scrub:**
```bash
zpool scrub vault
```

**List snapshots:**
```bash
zfs list -t snapshot
```

**Rollback to snapshot (if I delete something):**
```bash
# Access snapshots directly:
ls /mnt/vault/.zfs/snapshot/

# Or rollback entire dataset:
zfs rollback vault/mom@daily-2026-01-15
```

**Restic backup:**
```bash
restic -r s3:s3.amazonaws.com/your-bucket backup /mnt/vault
```

**Restic restore:**
```bash
restic -r s3:s3.amazonaws.com/your-bucket restore latest --target /mnt/restore
```

---

## Cost Summary

| What | One-Time | Ongoing |
|------|----------|---------|
| DAS + 4×8TB drives | ~$720-750 | — |
| UPS (battery backup) | ~$100-180 | — |
| M-Discs | ~$125-140 | ~$50/year for updates |
| AWS Glacier Deep Archive | — | ~$4-8/month |
| **Totals** | **~$945-1,070** | **~$50-100/year** |

---

## What You're Protected Against

| Threat | Protection |
|--------|------------|
| Drive failure | RAID-Z2 (survives 2 drives failing) |
| Bit rot / silent corruption | ZFS checksums + self-healing |
| Accidental deletion | ZFS snapshots + Glacier versions |
| Ransomware | Glacier is immutable, M-Discs are write-once |
| House fire / flood | Glacier (offsite) + M-Discs (at boyfriend's) |
| My own mistakes | Snapshots, versions, multiple copies |
| EMP / solar flare / collapse | M-Discs (inorganic, no electronics needed to store) |
| Total catastrophe | M-Discs readable on any Blu-ray drive for 100+ years |

---

## The 3-2-1 Rule: Achieved ✓

- **3 copies:** NAS (primary) + Glacier (cloud) + M-Discs (physical)
- **2 media types:** Hard drives + optical discs + cloud
- **1 offsite:** Glacier (always) + M-Discs at boyfriend's

---

## Notes

```
- RAM crisis (thanks AI boom) made used servers cost 3-4x normal
- Gaming/AI PC is more powerful than any NAS anyway
- Partner has full A/V digitization setup — VHS road trip planned
- Already own a BDXL burner somewhere

```

---

*Document created: January 2026*
*Last updated: January 2026*
*Reason for DAS approach: Sam Altman ate all the RAM*
