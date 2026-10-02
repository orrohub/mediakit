#  ORRO System Design — Open Source Update

Keeping with our nature of sharing what we do here's the diagram for own simplified system design for Beta launch

### Redundant dual node system architecture diagram

                                 ┌────────────────────────┐
                                 │     Load Balancer      │  ← health-checks both nodes,
                                 │                        │    routes traffic to whichever's up
                                 └───────────┬────────────┘
                            ┌────────────────┴────────────────┐
                            ▼                                  ▼
                     ┌─────────────┐                    ┌─────────────┐
                     │   Node #1   │                    │   Node #2   │   identical code —
                     │ (API + queue│                    │ (API + queue│   either can vanish
                     │   worker)   │                    │   worker)   │   with zero loss of
                     └──────┬──────┘                    └──────┬──────┘   capability
                            │                                  │
                ┌───────────┼─────────────┬────────────────────┼───────────┐
                ▼           ▼             ▼                    ▼           ▼
          ┌──────────┐ ┌─────────┐  ┌───────────┐       ┌───────────┐   (same 4 targets,
          │ Postgres │ │  Redis  │  │Blockchain │       │Blockchain │    reachable from
          │  (single │ │ (queue, │  │  API #1   │       │  API #2   │    both nodes)
          │  source  │ │ risk    │  └───────────┘       └───────────┘
          │ of truth)│ │ monitor,│
          └────┬─────┘ │  rate   │
               │       │ limits) │
               │       └─────────┘
               ▼
         ┌──────────────┐
         │   Backups    │  ← runs on its own schedule, straight against
         │ (WAL/pg_dump)│     Postgres — never depends on either node
         └──────────────┘
