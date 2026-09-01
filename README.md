# Delhi Metro: Does Train Frequency Match Network Importance?

An analysis of DMRC's published timetable asking whether the service each
station receives reflects how important that station is to the network.

**Result:** 33 of 239 stations receive less service than their structural role
warrants. 12 of them are on the Red Line.

**Stack:** Python (pandas, networkx) · SQL (SQLite) · Power BI

---

## The question

DMRC sets train frequency line by line. But passengers move through the
network as a graph, not as a set of separate lines.

Some stations sit on many of the shortest paths between other stations — they
carry through-traffic from across the system. Others sit near the end of a
branch and mainly serve their own catchment.

**If frequency is allocated purely by line, are some structurally important
stations receiving less service than their role warrants?**

---

## Data

DMRC static GTFS feed, published on the Delhi Open Transit Data portal
(https://otd.delhi.gov.in/data/staticDMRC), feed dated 10 August 2023.

| | |
|---|---|
| Stations | 262 (239 after excluding the disconnected Aqua Line) |
| Routes | 36 (18 lines × 2 directions) |
| Scheduled stop events | 128,434 total, 126,723 on the weekday timetable |
| Service span | 04:45 to 01:13 the following day |

Data quality was clean: no orphan records, no missing coordinates.

---

## Method

**Service supply.** Each `stop_times` row is one train calling at one station.
Counting those within a time band and dividing by band length gives trains per
hour per direction — what a passenger on the platform experiences.

**Network importance.** Weighted betweenness centrality on the station graph,
with edge weights set to median observed run times between adjacent stations.
A high-betweenness station is one whose loss would force large detours.

**The mismatch index.** Both measures converted to percentile ranks, then
differenced. A positive gap means the station matters more to the network than
its frequency reflects.

### Three data issues worth naming

1. **Interchanges appear multiple times.** GTFS assigns a separate `stop_id`
   per line, so Kashmere Gate appears three times. These are merged on a
   normalised station name. Skipping this step materially distorts centrality.

2. **`direction_id` is blank in this feed.** Direction is encoded in the route
   instead — each line appears twice, the reverse carrying an `_R` suffix.
   Counting `route_id` rather than the derived line name would double every
   station's line count.

3. **Three timetables are bundled together** (weekday, saturday, sunday).
   Mixing them inflates apparent frequency. The analysis uses weekday only.

---

## Findings

### 1. Peak frequency varies fourfold by line

Weekday 07:00–11:00, trains per hour per direction at an average station:

| Line | Trains/hr | Wait | Range across stations |
|---|---|---|---|
| Yellow | 19.6 | 3.1 min | 8.7–45.3 |
| Blue | 18.4 | 3.3 min | 10.1–37.2 |
| Violet | 13.8 | 4.3 min | 7.7–26.6 |
| Green | 12.6 | 4.8 min | 6.4–13.0 |
| Rapid | 11.6 | 5.2 min | 11.2–11.7 |
| Magenta | 10.6 | 5.7 min | 10.5–10.7 |
| Red | 10.5 | 5.7 min | 9.0–20.1 |
| Pink | 10.5 | 5.7 min | 10.2–10.8 |
| Orange (Airport) | 6.0 | 10.0 min | 6.0 |
| Gray | 4.7 | 12.8 min | 4.7–4.8 |

### 2. Supply and importance are correlated, but loosely

Spearman correlation between peak frequency and betweenness centrality:
**0.53**. Positive, so allocation is not arbitrary. Far from 1.0, so there are
real exceptions.

| | Stations |
|---|---|
| Service matches network role | 165 |
| Service exceeds network role | 41 |
| Underserved relative to role | 33 |

### 3. The Red Line is the clearest mismatch

12 of the 33 underserved stations are on the Red Line — and the Red Line has
only 28 stations, so nearly half of it is flagged. Shastri Park, Seelam Pur,
Tis Hazari, Pratap Nagar, Pul Bangash and Shahdara all appear near the top.

The Red Line runs east–west across north Delhi and carries through-traffic,
yet an average Red Line station gets 10.5 trains per hour against Yellow's
19.6.

**Recommendation:** on network-structure grounds, the Red Line is the
strongest candidate for a frequency increase.

---

## Validation

The centrality measure was checked against Delhi's known interchanges. The top
six by betweenness are Rajiv Chowk, Kashmere Gate, Central Secretariat, Hauz
Khas, Dilli Haat–INA and Patel Chowk — the network's genuine transfer hubs.
That the measure recovers them without being told about them is the main
evidence it is behaving correctly.

### A correction made during the analysis

An earlier draft reported Yellow Line frequency as 33.9 trains per hour by
counting every train dispatched on the line. That figure is wrong for any
statement about passenger waiting time, because the Yellow Line runs three
route variants with different endpoints and no single station is served by all
of them.

The per-station figure (19.6) is the correct basis. The wide station ranges in
the table above are what exposed the error: Yellow varies from 8.7 trains per
hour at outer stations to 45.3 where all variants overlap.

---

## Limitations

**This does not measure crowding.** DMRC does not publish station-level
passenger counts, so nothing here shows which stations are busy. A
low-frequency station may be perfectly comfortable if few people use it. All
findings concern supply and network position only. A demand-side extension
would need data DMRC does not release.

**Betweenness reflects topology, not population.** A station serving a dense
residential catchment but sitting at the network edge scores low. The measure
captures through-movement importance, not how many people live nearby.

**Scheduled, not actual.** GTFS is the published timetable. Real headways
differ through delays and short-turning.

**Accessibility excludes wait time.** The 45-minute reachability figure counts
in-vehicle time plus a flat 4-minute interchange penalty, ignoring the wait for
the first train. This overstates reachability at low-frequency stations — a
bias working against the very stations the analysis flags, so the underserved
list is if anything conservative.

**Aqua Line excluded.** 21 stations formed a separate graph component. These
are Noida Metro, operated by NMRC, with no track connection to DMRC.

**Feed vintage.** August 2023, predating later Phase IV extensions.


