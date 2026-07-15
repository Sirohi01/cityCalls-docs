# Rohit 07 — Form Field Specifications

Instance-by-instance field specs, using the template from §4 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md). Filled in per form as each module is built (per [13-ui-development-sequence.md](13-ui-development-sequence.md)) — no field is added to a screen ad hoc without a row here first. This document starts with the highest-priority forms (Phase 1-2) fully specified; later forms are added incrementally.

## Service Request Booking Form (Customer App + Admin "Create Service Request")

| Field key | Label | Input type | Data type | Required | Validation | Placeholder | Default | Master source | Visible when | Editable when | Role permission | API field | DB field |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `serviceId` | Service | Select (searchable) | ObjectId | Yes | active service only | "Select a service" | — | `GET /services?active=true` | always | create only | `serviceRequests.create` | `serviceId` | `service_requests.serviceId` |
| `customerProductId` | Appliance | Select or "Add new" | ObjectId | Yes | must belong to customer, or inline-create | "Select your appliance" | — | `GET /customers/{id}/products` | after service selected | create only | `serviceRequests.create` | `customerProductId` | `service_requests.customerProductId` |
| `addressId` | Service Address | Select or "Add new" | ObjectId | Yes | must belong to customer, or inline-create | "Select address" | default address | `customer.addresses[]` | always | create only | `serviceRequests.create` | `addressId` | `service_requests.addressSnapshot` |
| `symptoms` | What's wrong? | Multi-select checklist | ObjectId[] | Yes (min 1) | at least one selected | — | — | `GET /masters/SYMPTOM?serviceId=` | after service selected | create only | `serviceRequests.create` | `symptoms` | `service_requests.symptoms` |
| `notes` | Additional details | Textarea | string | No | max 500 chars | "Describe the issue further (optional)" | — | — | always | create only | `serviceRequests.create` | `notes` | `service_requests.notes` |
| `images` | Upload photos/video | File upload (multi) | ObjectId[] (file refs) | No, unless service requires | type/size per [10-api-standards.md](../10-api-standards.md) §9 | — | — | — | always | create only | `serviceRequests.create` | `images` | linked via file service |
| `preferredSlot` | Preferred time | Date + slot picker | {date, slotId} | Yes | must be an available slot for branch | — | earliest available | `GET /appointment-slots` | after address selected (branch resolved) | create only | `serviceRequests.create` | `preferredSlot` | `service_requests.scheduledDate/Slot` |
| `priority` | Priority | Select (staff only) | enum | No | valid Priority enum | — | `NORMAL` | [coordination/06-naming-conventions.md](../coordination/06-naming-conventions.md) §4 | staff-created only | create only | `serviceRequests.create` | `priority` | `service_requests.priority` |

## Login Form (Admin Web)

| Field key | Label | Input type | Data type | Required | Validation | Placeholder | Default | Master source | Visible when | Editable when | Role permission | API field | DB field |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `identifier` | Email or Mobile | Text | string | Yes | valid email or mobile format | "Email or mobile number" | — | — | always | always | public | `identifier` | `users.email`/`mobile` |
| `password` | Password | Password | string | Yes | min 8 chars | "Password" | — | — | always | always | public | `password` | — (hash compared) |

## Remaining Forms

Every other form (Customer create/edit, Vendor onboarding, Call entry per type, Lead capture, Estimate line-item entry, Service catalog editor, etc.) follows this exact table structure — added here as each is built, in the order set by [13-ui-development-sequence.md](13-ui-development-sequence.md), never designed ad hoc inside a component file.
