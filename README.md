# Backup-Backup
The backup to the backup


# Family Archive & Backup Plan — FINAL
### January 2026

---

## The Situation

Thanks to the AI boom sucking up all global RAM production, used servers that were $300-450 a year ago are now $1,200+, and even a basic Synology NAS doubled to $1,200+. So we're going with the smart play: my beefy gaming/AI rig does double duty as the storage server.

Thanks, Sam. 🙄

---

## What I'm Building

```
┌─────────────────────────────────────────────────────────────────┐
│                MY GAMING/AI PC (lives in my room)               │
│                                                                 │
│   RTX 4070 Ti │ 32GB RAM │ mergerFS + SnapRAID (or BTRFS)       │
│                                                                 │
│         ┌─────────────────────────────────────────┐             │
│         │            4-BAY DAS (USB 3.2)          │             │
│         │                                         │             │
│         │   4 × 8TB: 3 data + 1 parity = 24TB     │             │
│         │   SnapRAID checksums + parity recovery  │             │
│         │                                         │             │
│         │   Mom's data: ~750 GB                   │             │
│         │   My data: ~3 TB                        │             │
│         │   VHS/photo digitization: ~1.5 TB       │             │
│         │   Room to grow: ~18 TB                  │             │
│         └─────────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
         │
         │ Nightly (restic, runs while I sleep)
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                 AWS GLACIER DEEP ARCHIVE                        │
│                       ~$5/month                                 │
│                                                                 │
│   Encrypted, versioned, offsite                                 │
│   "Break glass" emergency backup (12-48hr retrieval)            │
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
| Growth headroom | ~18,700 GB |
| **Target** | **24 TB usable** (3 data drives) |

**Note on deduplication:** Do NOT use filesystem-level dedup (ZFS dedup needs ~5GB RAM per TB = 80GB+ for this pool). Instead, run file-level dedup with czkawka before storing. This achieves similar space savings with zero RAM overhead.

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

**Why I need a UPS:** Filesystems really don't like sudden power loss mid-write. A UPS gives the system time to flush writes and shut down cleanly during an outage. Also protects against surges.

---

## Filesystem Choice: Why mergerFS + SnapRAID

Given the RAM crisis and USB DAS setup, here's why we're NOT using ZFS:

### The Problem with ZFS + USB + Limited RAM

| Issue | Impact |
|-------|--------|
| ZFS wants 1GB RAM per TB | 16TB pool wants 16GB just for ZFS, leaving little for gaming/AI |
| TrueNAS VM adds overhead | +8GB for VM = 24GB total, only 8GB left for everything else |
| USB drives can disconnect | ZFS may mark drive as permanently faulted on USB hiccup |
| No ECC RAM (gaming rig) | ZFS + non-ECC has theoretical data corruption risk |

### Filesystem Comparison

| Factor | ZFS | BTRFS | mergerFS + SnapRAID |
|--------|-----|-------|---------------------|
| **RAM usage** | 8-16GB+ | 1-2GB | ~500MB |
| **USB tolerance** | Poor | Good | Excellent |
| **Real-time parity** | Yes | Yes | No (scheduled nightly) |
| **Drive failure recovery** | Immediate | Immediate | Next sync cycle |
| **Checksumming** | Yes | Yes | Yes (SnapRAID) |
| **Snapshots** | Excellent | Excellent | Manual/scripted |
| **Mixed drive sizes** | No | Limited | Yes |
| **Complexity** | High | Medium | Medium |
| **Reads individual drives** | No (pool only) | No (mostly) | Yes (each drive is ext4) |

### Why mergerFS + SnapRAID Wins Here

1. **USB drive drops off temporarily?** Other drives keep working, no array corruption
2. **RAM usage:** ~500MB total, leaves 31.5GB for gaming + AI
3. **Gaming PC busy?** Parity sync just waits until later
4. **Drive dies?** SnapRAID recovers it from parity
5. **Need to read drives on another PC?** Each drive is just ext4, readable anywhere
6. **Worst case (parity outdated)?** Glacier has everything anyway

### How mergerFS + SnapRAID Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        mergerFS                                 │
│   Pools multiple drives into one mount point (/mnt/vault)       │
│   No RAID - just presents all drives as one big folder          │
│   Files are stored on individual drives (configurable policy)   │
└─────────────────────────────────────────────────────────────────┘
                              +
┌─────────────────────────────────────────────────────────────────┐
│                        SnapRAID                                 │
│   Calculates parity across drives (runs nightly via cron)       │
│   Stores parity on dedicated drive(s)                           │
│   Can recover from 1-2 drive failures (depending on config)     │
│   Also does checksumming / bit rot detection                    │
└─────────────────────────────────────────────────────────────────┘

Example with 4 × 8TB drives:
  - Drive 1: 8TB data
  - Drive 2: 8TB data
  - Drive 3: 8TB data
  - Drive 4: 8TB PARITY
  - Total usable: 24TB (vs 16TB with RAID-Z2)
```

### Alternative: BTRFS RAID1

If you want real-time protection and are willing to sacrifice some space:

```
4 × 8TB in BTRFS RAID1 = 16TB usable
- Real-time mirroring (every write goes to 2 drives)
- Better than ZFS for USB tolerance
- 1-2GB RAM usage
- Survives any 1 drive failure (sometimes 2)
```

BTRFS RAID5/6 exists but is not recommended for critical data (historical bugs).

---

## Cloud Backup Comparison

### Why Glacier Deep Archive

This is a "break glass in emergency" backup. If my house burns down, waiting 12-48 hours for retrieval is fine — I'll be dealing with insurance and finding a place to stay anyway. The $100 retrieval fee for 5TB is nothing compared to losing everything.

| Factor | Glacier Deep Archive | Backblaze B2 |
|--------|---------------------|--------------|
| Storage cost (5TB) | **~$5/month** | ~$30/month |
| Retrieval cost | $0.02/GB ($100 for 5TB) | $0.00 |
| Retrieval time | 12-48 hours | Instant |
| Minimum storage | 180 days | None |
| restic compatible | Yes | Yes |
| **Annual cost** | **~$60/year** | ~$360/year |

**Bottom line:** $300/year savings. Retrieval time/cost only matters in a true emergency, and at that point $100 is irrelevant.

### Full Comparison Matrix

| Service | $/TB/month | Retrieval $/GB | Retrieval Time | Min Duration | restic? |
|---------|------------|----------------|----------------|--------------|---------|
| **Glacier Deep Archive** | **$0.99** | $0.02 | 12-48 hrs | 180 days | **Yes** |
| Glacier Flexible | $3.60 | $0.01 | 3-5 hrs | 90 days | Yes |
| Glacier Instant | $4.00 | $0.03 | Instant | 90 days | Yes |
| S3 Intelligent-Tiering | $0.99-4.00* | $0.00 | Instant-12hrs | None | Yes |
| Backblaze B2 | $6.00 | $0.00 | Instant | None | Yes |
| Wasabi | $7.00 | $0.00 | Instant | 90 days | Yes |

*S3 Intelligent-Tiering auto-moves data; reaches $0.99/TB after 180 days untouched

### Recommendation

**Primary choice: Glacier Deep Archive** — Cheapest option, ~$5/month for 5TB. Retrieval is slow but this is emergency-only backup.

**If budget allows later: Backblaze B2** — $30/month for instant retrieval. Nice to have, not essential when you have local + M-Disc copies.

---

## Grand Total

| Category | Cost |
|----------|------|
| DAS + drives | $720-750 |
| M-Discs | $125-140 |
| UPS | $100-180 |
| **One-time total** | **$945-1,070** |
| Glacier Deep Archive (annual) | ~$60/year |

**Money saved vs. alternatives (RAM crisis pricing):**
- vs. Synology DS923+ (~$1,200 + $600 drives): **~$850+ saved**
- vs. Used R720 ($1,200+ + $600 drives): **~$850+ saved**
- vs. Backblaze B2 instead of Glacier: **~$300/year saved**

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

First, create a secrets file at `~/.config/backup-secrets` (chmod 600!):

```bash
# ~/.config/backup-secrets
# NEVER commit this file to git!

# AWS credentials (for Glacier Deep Archive via S3)
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_DEFAULT_REGION=us-east-1

# Restic encryption password
export RESTIC_PASSWORD=your-strong-restic-password

# Healthchecks.io ping URL (optional but recommended)
export HEALTHCHECK_URL=https://hc-ping.com/your-unique-id

# iCloud credentials (optional - can also use keyring)
export ICLOUD_USERNAME=moms-apple-id@email.com
```

Set permissions:
```bash
chmod 600 ~/.config/backup-secrets
```

Now create the sync script at `/mnt/vault/system/scripts/daily-sync.sh`:

```bash
#!/bin/bash
# Daily cloud sync script
# Runs overnight, syncs all cloud sources, then backs up to Glacier

set -euo pipefail  # Exit on error, undefined vars, pipe failures

LOG=/var/log/daily-sync.log
VAULT=/mnt/vault
RESTIC_REPO=s3:s3.amazonaws.com/your-bucket-name
FAILED=0

# Load secrets from protected file
source ~/.config/backup-secrets

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> $LOG
}

alert_failure() {
    log "FAILURE: $1"
    if [ -n "${HEALTHCHECK_URL:-}" ]; then
        curl -fsS --retry 3 "${HEALTHCHECK_URL}/fail" -d "$1" || true
    fi
}

log "=============================="
log "Sync started"
log "=============================="

# 1. Sync my Google Drive
log "Starting Google Drive sync..."
if ! rclone sync gdrive: $VAULT/you/google-drive/ \
    --log-file=$LOG \
    --log-level INFO \
    --bwlimit 10M; then
    alert_failure "Google Drive sync failed"
    FAILED=1
fi

# 2. Sync mom's iCloud
log "Starting iCloud sync..."
if ! icloudpd --directory $VAULT/mom/icloud/ \
    --username "$ICLOUD_USERNAME" \
    --size original \
    --until-found 50 \
    >> $LOG 2>&1; then
    alert_failure "iCloud sync failed - may need re-authentication"
    FAILED=1
fi

# 3. Sync mom's Google Photos
log "Starting Google Photos sync..."
if ! gphotos-sync $VAULT/mom/google-photos/katieogill/ >> $LOG 2>&1; then
    alert_failure "Google Photos sync failed"
    FAILED=1
fi

# 4. Backup everything to S3 (lifecycle rule moves to Glacier Deep Archive)
log "Starting Glacier backup..."
if ! restic -r $RESTIC_REPO \
    backup $VAULT/mom $VAULT/you \
    --exclude-caches \
    >> $LOG 2>&1; then
    alert_failure "Glacier backup failed"
    FAILED=1
fi

# 5. Prune old snapshots
log "Pruning old snapshots..."
if ! restic -r $RESTIC_REPO \
    forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune \
    >> $LOG 2>&1; then
    alert_failure "Snapshot pruning failed"
    FAILED=1
fi

# 6. Report success (if nothing failed)
if [ $FAILED -eq 0 ]; then
    log "Sync complete - all tasks successful!"
    if [ -n "${HEALTHCHECK_URL:-}" ]; then
        curl -fsS --retry 3 "$HEALTHCHECK_URL" || true
    fi
else
    log "Sync complete - some tasks failed (see above)"
fi

exit $FAILED
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

**Option 2: Email on failure** (already built into the script above via healthchecks.io)

**Option 3: Healthchecks.io (free, recommended)**

1. Sign up at healthchecks.io
2. Create a check, get your unique URL
3. Add to `~/.config/backup-secrets`:
```bash
export HEALTHCHECK_URL=https://hc-ping.com/your-unique-id
```

The script will ping on success and `/fail` on failure. If no ping arrives on schedule, you get an email/push alert.

---

### Storage Health Monitoring

Add a cron job to check for storage problems (works with mergerFS+SnapRAID, BTRFS, or ZFS):

```bash
# /mnt/vault/system/scripts/storage-health.sh
#!/bin/bash

source ~/.config/backup-secrets
ALERT_EMAIL="you@email.com"

# For SnapRAID: check for sync errors
if command -v snapraid &> /dev/null; then
    if snapraid status 2>&1 | grep -q "DANGER\|error"; then
        echo "SnapRAID reports errors!" | mail -s "⚠️ STORAGE ALERT" $ALERT_EMAIL
        [ -n "${HEALTHCHECK_URL:-}" ] && curl -fsS "${HEALTHCHECK_URL}/fail" -d "SnapRAID error"
    fi
fi

# For BTRFS: check for errors
if mount | grep -q "type btrfs"; then
    if dmesg | tail -100 | grep -qi "btrfs.*error"; then
        echo "BTRFS errors in dmesg!" | mail -s "⚠️ STORAGE ALERT" $ALERT_EMAIL
        [ -n "${HEALTHCHECK_URL:-}" ] && curl -fsS "${HEALTHCHECK_URL}/fail" -d "BTRFS error"
    fi
fi

# For ZFS (if you go that route): check pool status
if command -v zpool &> /dev/null; then
    if zpool status 2>/dev/null | grep -qE "DEGRADED|FAULTED|OFFLINE|UNAVAIL"; then
        echo "ZFS pool unhealthy!" | mail -s "⚠️ STORAGE ALERT" $ALERT_EMAIL
        [ -n "${HEALTHCHECK_URL:-}" ] && curl -fsS "${HEALTHCHECK_URL}/fail" -d "ZFS pool error"
    fi
fi

# Check SMART status on all drives
for drive in /dev/sd?; do
    if smartctl -H "$drive" 2>/dev/null | grep -q "FAILED"; then
        echo "SMART failure on $drive!" | mail -s "⚠️ DRIVE FAILING" $ALERT_EMAIL
        [ -n "${HEALTHCHECK_URL:-}" ] && curl -fsS "${HEALTHCHECK_URL}/fail" -d "SMART failure $drive"
    fi
done
```

Add to cron (run daily):
```bash
crontab -e
# Add:
0 6 * * * /mnt/vault/system/scripts/storage-health.sh
```

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
- [ ] Install mergerFS and SnapRAID
- [ ] Format drives as ext4, mount individually
- [ ] Configure mergerFS pool (/mnt/vault)
- [ ] Configure SnapRAID (3 data + 1 parity)
- [ ] Run initial SnapRAID sync
- [ ] Connect UPS, configure auto-shutdown on power loss
- [ ] Test write speeds, verify setup

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
- [ ] Set lifecycle rule: transition to Glacier Deep Archive after 1 day
- [ ] Create IAM user with S3 access, get access keys
- [ ] Create secrets file (~/.config/backup-secrets)
- [ ] Install restic
- [ ] Initialize restic repo pointing to S3
- [ ] Run first full backup (will take a while!)
- [ ] Set up nightly cron job
- [ ] Set up healthchecks.io monitoring
- [ ] Test restore of a few random files
- [ ] Create manifest document

**Restic + S3/Glacier quick start:**
```bash
# Set up credentials
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export RESTIC_PASSWORD=your-encryption-password

# Initialize repo
restic -r s3:s3.amazonaws.com/your-bucket-name init

# Run backup
restic -r s3:s3.amazonaws.com/your-bucket-name backup /mnt/vault/mom /mnt/vault/you

# Check snapshots
restic -r s3:s3.amazonaws.com/your-bucket-name snapshots

# Restore (if ever needed - remember 12-48hr retrieval time for Glacier)
restic -r s3:s3.amazonaws.com/your-bucket-name restore latest --target /mnt/restore
```

**Important:** The S3 lifecycle rule automatically moves data to Glacier Deep Archive. restic writes to S3, AWS handles the rest.

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
| SnapRAID sync | Nightly | `snapraid sync` (via cron) |
| SnapRAID scrub | Weekly | `snapraid scrub -p 10` (scrubs 10% of data) |
| SnapRAID status | Weekly | `snapraid status` (check for errors) |
| SMART check | Weekly | `smartctl -a /dev/sdX` or via storage-health.sh |
| Check Glacier backup logs | Weekly | `restic snapshots` (see Quick Reference) |
| Test restore from Glacier | Quarterly | Restore a few random files (allow 12-48hr retrieval) |
| M-Disc spot check | Annually | Read a random disc, verify files open |
| Update M-Disc archive | Annually | Burn new discs for major additions |

---

## Security & "Don't Nuke Anything" Checklist

- [ ] **SnapRAID parity protects data** — can recover from drive failures
- [ ] **SnapRAID checksums detect bit rot** — silent corruption caught
- [ ] **Glacier backups are versioned** — deletions don't propagate
- [ ] **restic encrypts everything** — even AWS can't read your data
- [ ] **Secrets file is chmod 600** — credentials not world-readable
- [ ] **Separate user permissions** — mom's folder vs mine
- [ ] **UPS protects against power events** — no mid-write corruption
- [ ] **M-Discs are write-once** — can't accidentally overwrite
- [ ] **Manifest in 3 places** — NAS, cloud, printed
- [ ] **Healthchecks.io alerts** — know when backups fail

---

## Quick Reference Commands

**SnapRAID status:**
```bash
snapraid status          # Check array health
snapraid smart           # Check drive SMART status
snapraid diff            # See what changed since last sync
```

**SnapRAID sync (run nightly):**
```bash
snapraid sync            # Update parity
```

**SnapRAID scrub (run weekly):**
```bash
snapraid scrub -p 10     # Scrub 10% of data, verify checksums
```

**Recover from drive failure:**
```bash
# Replace failed drive, format as ext4, mount
snapraid fix -d disk1    # Rebuild disk1 from parity
```

**mergerFS check:**
```bash
df -h /mnt/vault         # Check combined pool space
ls /mnt/disk*            # Check individual drives
```

**Restic backup:**
```bash
source ~/.config/backup-secrets
restic -r s3:s3.amazonaws.com/your-bucket-name backup /mnt/vault
```

**Restic restore:**
```bash
# Note: Glacier Deep Archive has 12-48hr retrieval time
# You may need to restore objects to S3 first via AWS console
source ~/.config/backup-secrets
restic -r s3:s3.amazonaws.com/your-bucket-name restore latest --target /mnt/restore
```

**Restic snapshots:**
```bash
source ~/.config/backup-secrets
restic -r s3:s3.amazonaws.com/your-bucket-name snapshots
```

---

## Cost Summary

| What | One-Time | Ongoing |
|------|----------|---------|
| DAS + 4×8TB drives | ~$720-750 | — |
| UPS (battery backup) | ~$100-180 | — |
| M-Discs | ~$125-140 | ~$50/year for updates |
| Glacier Deep Archive (~5TB) | — | ~$5/month |
| Healthchecks.io | — | Free |
| **Totals** | **~$945-1,070** | **~$60-110/year** |

*Note: Glacier has 12-48hr retrieval and ~$100 fee for 5TB restore. Acceptable for emergency-only backup.*

---

## What You're Protected Against

| Threat | Protection |
|--------|------------|
| Drive failure | SnapRAID parity (survives 1 drive failing) |
| Bit rot / silent corruption | SnapRAID checksums + scrub |
| Accidental deletion | Glacier versioning (restic keeps snapshots) |
| Ransomware | Glacier is offsite, M-Discs are write-once |
| House fire / flood | Glacier (offsite) + M-Discs (at boyfriend's) |
| My own mistakes | Glacier snapshots, M-Disc copies |
| USB drive disconnects | mergerFS gracefully handles it (other drives still work) |
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
- Using mergerFS + SnapRAID instead of ZFS due to RAM constraints and USB DAS
- Glacier Deep Archive for cloud (~$5/month) — emergency-only, 12-48hr retrieval is fine
- No ZFS dedup (would need 80GB+ RAM for 16TB)
- Synology NAS prices doubled due to RAM crisis — DIY is the only sane option
```

---

*Document created: January 2026*
*Last updated: January 2026*
*Reason for DAS approach: Sam Altman ate all the RAM*
*Reason for mergerFS over ZFS: Sam Altman also ate all the RAM we'd need for ZFS*
