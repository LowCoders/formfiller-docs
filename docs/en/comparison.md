# Comparisons

This document compares the FormFiller system with other approaches and services.

## Table of Contents

1. [Architecture Comparison (MVC/MVP vs FormFiller)](#architecture-comparison)
2. [Form Builder Services Comparison](#form-builder-comparison)
3. [Summary Evaluation](#summary-evaluation)

---

## Architecture Comparison

### Problems with Traditional MVC/MVP Systems

Classic Model-View-Controller (MVC) or Model-View-Presenter (MVP) architectures result in significant redundancy:

#### Redundancy Problem

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MVC/MVP Architecture - Redundancy                     │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                         DATABASE LAYER                              │ │
│  │  • users.sql - CREATE TABLE users (name VARCHAR, email VARCHAR...)  │ │
│  │  • Migration management                                             │ │
│  │  • Indexes, constraints                                             │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                               ↓ Duplication                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                          BACKEND LAYER                              │ │
│  │  • User.model.ts - interface User { name: string, email: string }   │ │
│  │  • UserDTO.ts - class UserDTO { @IsString() name, @IsEmail() email }│ │
│  │  • user.validation.ts - Joi/Yup schema                              │ │
│  │  • user.controller.ts - CRUD endpoints                              │ │
│  │  • user.service.ts - Business logic                                 │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                               ↓ Duplication                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                         FRONTEND LAYER                              │ │
│  │  • user.types.ts - interface User (again!)                          │ │
│  │  • UserForm.tsx - <input name="name"/> <input name="email"/>        │ │
│  │  • user.validation.ts - Frontend validation (again!)                │ │
│  │  • user.api.ts - API calls                                          │ │
│  │  • user.store.ts - State management                                 │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

#### Concrete Numbers

A simple "User" entity in traditional MVC:

| File/Component | Lines (approx.) | Purpose |
|----------------|-----------------|---------|
| DB migration | 20 | Table creation |
| Backend Model | 30 | TypeScript interface |
| Backend DTO | 40 | Validation decorators |
| Backend Controller | 80 | CRUD endpoints |
| Backend Service | 60 | Business logic |
| Frontend Types | 30 | Interface (duplicated!) |
| Frontend Form | 150 | React component |
| Frontend Validation | 40 | Validation (duplicated!) |
| Frontend API | 50 | HTTP calls |
| Frontend Store | 60 | State management |
| **Total** | **~560 lines** | |

### FormFiller Solution

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FormFiller Architecture - Zero Redundancy             │
│                                                                          │
│                    ┌─────────────────────────────────┐                   │
│                    │      FormFiller Schema          │                   │
│                    │     (single JSON file)          │                   │
│                    │                                 │                   │
│                    │  {                              │                   │
│                    │    "name": "userForm",          │                   │
│                    │    "items": [                   │                   │
│                    │      { "name": "name",          │                   │
│                    │        "type": "text",          │                   │
│                    │        "validationRules": [...] │                   │
│                    │      },                         │                   │
│                    │      { "name": "email",         │                   │
│                    │        "type": "text",          │                   │
│                    │        "validationRules": [...] │                   │
│                    │      }                          │                   │
│                    │    ]                            │                   │
│                    │  }                              │                   │
│                    └──────────────┬──────────────────┘                   │
│                                   │                                      │
│              ┌────────────────────┼────────────────────┐                 │
│              ↓                    ↓                    ↓                 │
│       ┌────────────┐       ┌────────────┐       ┌────────────┐          │
│       │  MongoDB   │       │  Backend   │       │  Frontend  │          │
│       │ (auto-save)│       │ (auto-API) │       │ (auto-UI)  │          │
│       └────────────┘       └────────────┘       └────────────┘          │
└─────────────────────────────────────────────────────────────────────────┘
```

#### Same "User" form in FormFiller:

```json
{
  "name": "userForm",
  "title": "User Registration",
  "items": [
    {
      "name": "name",
      "type": "text",
      "label": "Full Name",
      "validationRules": [
        { "type": "required", "message": "Required field" },
        { "type": "stringLength", "min": 2, "max": 100 }
      ]
    },
    {
      "name": "email",
      "type": "text",
      "label": "Email Address",
      "validationRules": [
        { "type": "required" },
        { "type": "email", "message": "Invalid email format" }
      ]
    }
  ]
}
```

**Total: ~25 lines** (vs. 560 lines in traditional MVC)

### Comparison Table

| Aspect | Traditional MVC | FormFiller |
|--------|-----------------|------------|
| **Definition locations** | 4-6 (DB, Model, DTO, Controller, Form, Store) | 1 (Schema) |
| **Lines of code (simple form)** | ~500-600 | ~25-50 |
| **Adding new field** | 6+ file modifications | 1 file (or UI) |
| **Validation consistency** | Manually synchronized | Automatically guaranteed |
| **Type safety** | Manual maintenance | Generated from schema |
| **Modification risk** | High (many touch points) | Low (single point) |
| **Learning curve** | High (many technologies) | Medium (JSON + rules) |
| **Development speed** | Days | Hours/Minutes |

---

## Form Builder Comparison

### Compared Systems

| | Google Forms | Microsoft Forms | Typeform | JotForm | FormFiller |
|---|:---:|:---:|:---:|:---:|:---:|
| **Price** | Free | Free* | Freemium | Freemium | Open Source |
| **Self-hosted** | No | No | No | No | **Yes** |

*With Microsoft 365 subscription

### Feature Comparison

#### Basic Features

| Feature | Google Forms | MS Forms | Typeform | JotForm | FormFiller |
|---------|:---:|:---:|:---:|:---:|:---:|
| Drag & Drop editor | Yes | Yes | Yes | Yes | Planned |
| Templates | Yes | Yes | Yes | Yes | Yes |
| Mobile-friendly | Yes | Yes | Yes | Yes | Yes |
| Embedding | Yes | Yes | Yes | Yes | Yes |
| Multi-language | Partial | Partial | No | Yes | **Full** |

#### Advanced Features

| Feature | Google Forms | MS Forms | Typeform | JotForm | FormFiller |
|---------|:---:|:---:|:---:|:---:|:---:|
| Conditional logic | Basic | Basic | Good | Good | **Excellent** |
| Calculated fields | No | No | No | Yes | **Yes** |
| Validation customization | Basic | Basic | Basic | Good | **Excellent** |
| File upload | Yes | Yes | Yes | Yes | Yes |
| Digital signature | No | No | No | Yes | Planned |

#### Data Management

| Feature | Google Forms | MS Forms | Typeform | JotForm | FormFiller |
|---------|:---:|:---:|:---:|:---:|:---:|
| Response export | CSV, Sheets | Excel | CSV | Multiple formats | **JSON, CSV, Excel** |
| API access | Limited | Limited | Yes | Yes | **Full REST API** |
| Webhook | No | Power Automate | Yes | Yes | **Yes** |
| Database integration | Sheets | Excel | No | MySQL, etc. | **MongoDB native** |

#### Workflow and Automation

| Feature | Google Forms | MS Forms | Typeform | JotForm | FormFiller |
|---------|:---:|:---:|:---:|:---:|:---:|
| Email notifications | Basic | Basic | Yes | Yes | **Customizable** |
| Workflow engine | No | Power Automate* | No | Basic | **Built-in** |
| Approval process | No | No | No | Yes | **Yes** |
| Automatic actions | No | Power Automate* | Zapier | Yes | **Native** |

*Separate product/subscription

#### Security and Compliance

| Feature | Google Forms | MS Forms | Typeform | JotForm | FormFiller |
|---------|:---:|:---:|:---:|:---:|:---:|
| GDPR compliance | Yes | Yes | Yes | Yes | **Full control** |
| Data location | Google Cloud | Azure | AWS | Variable | **Own server** |
| SSO integration | Google | Azure AD | No | Yes | **LDAP, OAuth, SAML** |
| Audit log | No | Partial | No | Yes | **Detailed** |
| Role management | Basic | Basic | No | Yes | **RBAC** |

#### Developer Features

| Feature | Google Forms | MS Forms | Typeform | JotForm | FormFiller |
|---------|:---:|:---:|:---:|:---:|:---:|
| API documentation | Limited | Limited | Good | Good | **Full (Swagger)** |
| SDKs | No | No | Yes | Yes | **TypeScript** |
| Custom components | No | No | No | Yes | **Yes** |
| White-label | No | No | Paid | Yes | **By default** |
| Source code access | No | No | No | No | **Full** |

### FormFiller Unique Advantages

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FormFiller vs SaaS Form Builders                      │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ 1. FULL CONTROL                                                     │ │
│  │    • Runs on your own server (on-premise, cloud, hybrid)            │ │
│  │    • Data never leaves your organization                            │ │
│  │    • No vendor lock-in                                              │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ 2. UNLIMITED CUSTOMIZATION                                          │ │
│  │    • Source code modifiable                                         │ │
│  │    • Custom components developable                                  │ │
│  │    • Any integration achievable                                     │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ 3. ADVANCED VALIDATION                                              │ │
│  │    • Group validators (field and form level)                        │ │
│  │    • ComputedRules (e.g., exam scoring)                             │ │
│  │    • Cross-field validation with full path support                  │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ 4. WORKFLOW ENGINE                                                  │ │
│  │    • Multi-step processes (validate → save → notify → api)          │ │
│  │    • Error handling strategies (stop, continue, rollback)           │ │
│  │    • Conditional execution                                          │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ 5. AI INTEGRATION                                                   │ │
│  │    • Form generation from natural language                          │ │
│  │    • Schema-based, validatable output                               │ │
│  │    • No extra cost (can use own LLM)                                │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ 6. MULTISITE CAPABILITY                                             │ │
│  │    • Multiple tenants on single installation                        │ │
│  │    • Optionally isolated databases                                  │ │
│  │    • Central management                                             │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

### When to Choose FormFiller?

| Requirement | SaaS (Google/MS/etc.) | FormFiller |
|-------------|:---------------------:|:----------:|
| Quick, simple surveys | Better choice | - |
| Occasional use | Better choice | - |
| Data protection requirements (GDPR, SOC2) | - | **Better choice** |
| Complex validation logic | - | **Better choice** |
| Workflow automation | - | **Better choice** |
| Custom UI/branding | - | **Better choice** |
| API-first approach | - | **Better choice** |
| On-premise installation | Not possible | **Better choice** |
| Source code access | Not possible | **Better choice** |

---

## Summary Evaluation

### Star Rating (1-5 ★)

| Category | Google Forms | MS Forms | Typeform | JotForm | FormFiller |
|----------|:------------:|:--------:|:--------:|:-------:|:----------:|
| **Ease of use** | ★★★★★ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| **Customizability** | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ |
| **Validation** | ★★☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★★ |
| **Workflow** | ★☆☆☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★★ |
| **API/Integration** | ★★☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★☆ | ★★★★★ |
| **Data protection** | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ |
| **Value for money** | ★★★★★ | ★★★★☆ | ★★★☆☆ | ★★★☆☆ | ★★★★★ |
| **Scalability** | ★★★★☆ | ★★★★☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ |
| **Developer experience** | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ |
| **Self-hosting** | ☆☆☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ☆☆☆☆☆ | ★★★★★ |
| | | | | | |
| **Total** | **24/50** | **25/50** | **28/50** | **36/50** | **47/50** |

### Rating Explanation

| Stars | Meaning |
|-------|---------|
| ★★★★★ | Excellent - Market leader or unique feature |
| ★★★★☆ | Very good - Above average solution |
| ★★★☆☆ | Good - Average, adequate |
| ★★☆☆☆ | Weak - Limited functionality |
| ★☆☆☆☆ | Minimal - Basic or missing |
| ☆☆☆☆☆ | Not available |

### Summary Table - Target Groups

| Target Group | Recommended Solution | Justification |
|--------------|---------------------|---------------|
| **Individuals, hobby** | Google Forms | Free, simple |
| **Small business (MS 365)** | Microsoft Forms | Integrated ecosystem |
| **Marketing, UX** | Typeform | Visually appealing |
| **SMB, custom needs** | JotForm | Good balance features/price |
| **Enterprise, developers** | **FormFiller** | Full control, customization |
| **GDPR-sensitive industries** | **FormFiller** | On-premise, data sovereignty |
| **Complex business logic** | **FormFiller** | Workflow, validation |

### Final Recommendation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         When to choose FormFiller?                       │
│                                                                          │
│  ✅ If you need to run on your own server (compliance, GDPR)            │
│  ✅ If complex validation and workflow is needed                        │
│  ✅ If API-first approach is required                                   │
│  ✅ If custom components or integrations need to be developed           │
│  ✅ If multisite/multi-tenant architecture is needed                    │
│  ✅ If you want AI-based form generation                                │
│  ✅ If you're looking for a long-term, maintainable solution            │
│                                                                          │
│  ❌ If you just need a quick, one-time survey                           │
│  ❌ If there's no technical resource for installation                   │
│  ❌ If visual drag & drop editor is critical (currently planned)        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Related Documentation

- [README](./index.md) - Project overview and motivation
- [Architecture](./architecture.md) - System structure
- [Enhancement Opportunities](./roadmap.md) - Development directions
- [Schema](./developer/schema.md) - Low-code definition language
- [Workflow](./developer/features/workflow.md) - Workflow management

