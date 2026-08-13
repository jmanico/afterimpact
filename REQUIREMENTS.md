# AfterImpact — Requirements

Car Accident Management Software. This document owns WHAT the system must do. Architecture, technology choices, deployment, and visual design are out of scope here.

## Required Requirement Inputs

- Project purpose: Assist a person who has had a recent car accident in managing associated paperwork, tracking health issues, and generally getting their life back on track.
- Primary users / actors: A person recovering from a recent car accident (the accident victim). No other actors are named in the notes.
- Core workflows: (1) Managing accident-related paperwork; (2) Tracking accident-related health issues; (3) Tracking overall recovery — getting life back on track after a major accident.
- Business objects / data entities: Accident, paperwork/documents, health issues. Further entities UNKNOWN.
- External integrations: UNKNOWN
- Authentication / roles: UNKNOWN
- Regulatory or privacy constraints: UNKNOWN (health-related data is involved, but the notes name no specific regulation — see OQ-5)

## Functional Requirements

### Accident Record

- **FR-1.1** The system MUST allow a user to record a car accident so that paperwork, health issues, and recovery items can be associated with it.

### Paperwork Management

- **FR-2.1** The system MUST allow a user to add accident-associated paperwork items to their accident record.
- **FR-2.2** The system MUST allow a user to view the paperwork items associated with their accident.
- **FR-2.3** The system SHOULD allow a user to update or remove a paperwork item they previously added.

### Health Issue Tracking

- **FR-3.1** The system MUST allow a user to record health issues resulting from their accident.
- **FR-3.2** The system MUST allow a user to view the health issues recorded for their accident.
- **FR-3.3** The system SHOULD allow a user to update a health issue as it changes over time (fields and status values TO BE DECIDED — see OQ-3).

### Recovery Tracking

- **FR-4.1** The system MUST help a user track their overall recovery after a major accident ("getting life back on track"); the concrete tasks or areas covered are TO BE DECIDED — see OQ-4.

## Open Questions

- **OQ-1** What forms does "paperwork" take — uploaded files/scans, structured data entry (e.g., insurance claims, police reports), deadlines/reminders, or a combination? This determines the required capabilities of FR-2.x and how they are tested.
- **OQ-2** Are there other actors besides the accident victim (e.g., a family member/helper, lawyer, insurance agent), and does the system need accounts, authentication, or role-based access at all?
- **OQ-3** What information must a health issue capture (symptoms, treatments, appointments, providers, status), and what is its lifecycle?
- **OQ-4** What does "get your life back on track" concretely include — task checklists, reminders, guidance content, financial tracking, vehicle repair/replacement? This defines the scope of FR-4.1.
- **OQ-5** Do any regulatory or privacy constraints apply (e.g., health-data or personal-data protection rules), given that the system stores health information?
- **OQ-6** Can a user manage more than one accident, or does the system assume a single recent accident per user?
- **OQ-7** Are external integrations expected (insurance companies, healthcare providers, document storage), or is all data entered manually by the user?
