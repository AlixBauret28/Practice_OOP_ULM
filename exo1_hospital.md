
# Exo 1 - Hospital Management System 🏥

## Task 1: Basic Class Diagram (3 classes)

```mermaid
classDiagram
    class Patient {
        -id: String
        -name: String
        -email: String
        -password: String
        +login()
        +postProblem()
        +payPrescription()
        +viewPrescription()
    }

    class Doctor {
        -id: String
        -name: String
        -specialty: String
        -password: String
        +login()
        +viewProblem()
        +prescribeSolution()
    }

    class Organizer {
        -id: String
        -name: String
        -password: String
        +login()
        +readProblem()
        +sendPrescription()
        +collectFee()
        +payDoctor()
        +createMember()
        +updateSystem()
    }

    Patient "1..*" -- "1" Organizer : contacts
    Organizer "1" -- "1..*" Doctor : advises
    Patient "1..*" -- "0..*" Doctor : consult via organizer
```

---

## Task 2: Complete Class Diagram (6 classes) + Sequence Diagram

### Class Diagram

```mermaid
classDiagram
    class User {
        <<abstract>>
        -id: String
        -name: String
        -email: String
        -password: String
        +login()
        +logout()
        +updateProfile()
    }

    class Doctor {
        -specialty: String
        -licenseNumber: String
        +viewProblem()
        +prescribeSolution()
        +viewPatientHistory()
    }

    class Organizer {
        -role: String
        +readProblem()
        +forwardToDoctor()
        +sendPrescription()
        +collectFee()
        +payDoctor()
        +createMember()
        +updateDatabase()
        +issueCredentials()
    }

    class Patient {
        -address: String
        -phone: String
        +postProblem()
        +viewPrescription()
        +makePayment()
        +viewHistory()
    }

    class HealthProblem {
        -id: String
        -description: String
        -datePosted: Date
        -status: ProblemStatus
        +updateStatus()
        +getDetails()
    }

    class Prescription {
        -id: String
        -diagnosis: String
        -medication: String
        -instructions: String
        -dateIssued: Date
        +getDetails()
    }

    class Payment {
        -id: String
        -amount: Double
        -date: date
        -status: PaymentStatus()
        +process()
        +getReceipt()
    }

    class CashPayment {
        -receivedBy: String
        +process()
    }

    class CheckPayment {
        -checkNumber: String
        -bankName: String
        +process()
    }

    class CreditCardPayment {
        -cardNumber: String
        -ExpiryDate: String
        +process()
        +validatecard()
    }

class ProblemStatus {
        <<enumeration>>
        PENDING
        IN_REVIEW
        RESOLVED
        CLOSED
    }

class PaymentStatus {
        <<enumeration>>
        PENDING
        COMPLETEED
        FAILED
    }

###inheritance

    User <|-- Patient
    User <|-- Doctor
    User <|-- Organizer

    Payment <|-- CashPayment
    Payment <|-- CheckPayment
    Payment <|-- CreditCardPayment


Patient "1" --> "0..*" HealthProblem : submits
Organizer "1" --> "0..*" HealthProblem : manages

Patient "1" --> "0..*" Payment : makes
Organizer "1" --> "0..*" Payment : processes
Payment "0..*" --> "1" Doctor : paid to

HealthProblem "1" --> "0..*" Prescription : receives
Doctor "1" --> "0..*" Prescription : creates

HealthProblem --> ProblemStatus
Payment --> PaymentStatus

```

### Sequence Diagram: Patient submits problem, receives prescription, and pays

```mermaid
sequenceDiagram
    actor P as Patient
    participant O as Organizer
    participant D as Doctor
    participant Sys as System

    P->>Sys: Submit health problem
    Sys->>O: Notify new problem
    O->>O: Read problem
    O->>D: Consult doctor with problem
    D->>D: Review problem
    D->>Sys: Create prescription
    Sys->>O: Notify prescription ready
    O->>P: Send prescription
    P->>Sys: Make payment (check/cash/card)
    Sys->>O: Notify payment received
    O->>D: Forward payment
```

---

## Task 3: Advanced - System Sequence Diagrams

### 1. Patient Registration

```mermaid
sequenceDiagram
    actor O as Organizer
    participant Sys as System
    actor P as Patient

    O->>Sys: createMember(name, email)
    Sys->>Sys: Generate ID and password
    Sys->>Sys: Save to database
    Sys-->>O: Confirm member created
    O->>P: Send member ID and password
    P->>Sys: Login with credentials
    Sys-->>P: Access granted
```

### 2. Consultation Flow

```mermaid
sequenceDiagram
    actor P as Patient
    participant Sys as System
    participant O as Organizer
    participant D as Doctor

    P->>Sys: Login
    P->>Sys: postProblem(description)
    Sys->>Sys: Save HealthProblem (status: pending)
    Sys->>O: Notify new problem
    O->>Sys: readProblem(problemId)
    Sys->>Sys: Update status (in-review)
    O->>D: consultDoctor(problemId)
    D->>Sys: reviewProblem(problemId)
    D->>Sys: createPrescription(problemId, content)
    Sys->>Sys: Save Prescription + timestamp
    Sys->>Sys: Update HealthProblem status (resolved)
    Sys->>O: Notify prescription ready
    O->>Sys: sendPrescription(patientId, prescriptionId)
    Sys->>P: Deliver prescription
```

### 3. Payment Processing

```mermaid
sequenceDiagram
    actor P as Patient
    participant Sys as System
    participant O as Organizer
    participant D as Doctor

    P->>Sys: selectPaymentMethod(type)
    alt Cash
        P->>Sys: submitCashPayment(amount)
    else Check
        P->>Sys: submitCheckPayment(checkNumber, amount)
    else Credit Card
        P->>Sys: submitCardPayment(cardDetails, amount)
        Sys->>Sys: Validate card
    end
    Sys->>Sys: Record payment + timestamp
    Sys-->>P: Payment confirmed
    Sys->>O: Notify payment received
    O->>Sys: forwardPayment(doctorId, amount)
    Sys->>D: Payment forwarded
    Sys-->>O: Transfer confirmed
```

---

## Questions to Consider

**1. How would you handle a patient consulting multiple doctors?**
The class diagram already supports this through the Organizer, who can consult multiple doctors for a single HealthProblem. You could add a many-to-many relationship between HealthProblem and Doctor if needed.

**2. Should prescriptions be reusable? How would you model that?**
Prescriptions should not be reusable as-is, since each one is tied to a specific health problem. However, a doctor could reference past prescriptions when creating a new one. This could be modeled with a self-referencing association on the Prescription class (e.g., `referencePrescription`).

**3. What design pattern fits the payment system?**
The Strategy pattern. Each payment type (Cash, Check, CreditCard) implements a common Payment interface with a `process()` method. This allows the system to handle different payment methods without changing the core logic.

**4. How do you ensure data integrity when payments are processed?**
Use transactions at the database level to ensure that payment recording and forwarding happen atomically. If any step fails, the entire operation is rolled back. Also add validation checks on amounts and status updates.

