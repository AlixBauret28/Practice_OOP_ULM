
# Exo 2 - ABCD Records 📼

## Level 1

### Task 1.1: Basic Use Case Diagram

```mermaid
graph LR
    subgraph ABCD Records System
        UC1((Place Order))
        UC2((Verify Membership))
        UC3((Process Order))
        UC4((Make Payment))
    end

    Member([Member]) --> UC1
    Member --> UC4
    OPC([Order Processing Clerk]) --> UC2
    OPC --> UC3
```

### Task 1.2: Basic Class Diagram

```mermaid
classDiagram
    class Member {
        -int memberId
        -String name
        -String address
        -String memberType
        +placeOrder()
        +makePayment()
        +viewOrders()
    }

    class Order {
        -int orderId
        -DateTime orderDate
        -String status
        -float totalAmount
        +calculateTotal()
        +updateStatus()
        +addItem()
    }

    class OrderProcessingClerk {
        -int clerkId
        -String name
        -String department
        +verifyMembership()
        +processOrder()
        +printInvoice()
    }

    Member "1" -- "0..*" Order : places
    OrderProcessingClerk "1" -- "0..*" Order : processes
```

---

## Level 2

### Task 2.1: Complete Use Case Diagram

```mermaid
graph LR
    subgraph ABCD Records System
        UC1((Place Order))
        UC2((Verify Membership))
        UC3((Process Order))
        UC4((Make Payment))
        UC5((Verify Item Availability))
        UC6((Apply Discount))
        UC7((Print Invoice))
        UC8((Print Shipping List))
        UC9((Order Unavailable Items))
        UC10((Send Membership Application))
    end

    Member([Member]) --> UC1
    Member --> UC4
    NonMember([Non-Member]) --> UC10
    OPC([Order Processing Clerk]) --> UC2
    OPC --> UC3
    OPC --> UC5
    OPC --> UC7
    OPC --> UC8
    CDC([Collection Dept Clerk]) --> UC4

    UC1 -.->|includes| UC2
    UC3 -.->|includes| UC5
    UC3 -.->|includes| UC7
    UC3 -.->|includes| UC8
    UC1 -.->|extends| UC6
    UC1 -.->|extends| UC9
```

### Task 2.2: Complete Class Diagram

```mermaid
classDiagram
    class Member {
        <<abstract>>
        -int memberId
        -String name
        -String address
        -String password
        +placeOrder()
        +makePayment()
        +viewOrders()
    }

    class RoyalMember {
        -float discountRate
        -boolean hasPriority
        +orderUnavailableItem()
        +getDiscount()
    }

    class RegularMember {
        -DateTime joinDate
        +upgradeMembership()
    }

    class Order {
        -int orderId
        -DateTime orderDate
        -String status
        -float totalAmount
        +calculateTotal()
        +updateStatus()
        +addItem()
    }

    class OrderLine {
        -int quantity
        -float lineTotal
        +calculateLineTotal()
    }

    class Item {
        <<abstract>>
        -int itemId
        -String title
        -String artist
        -float price
        -boolean available
        +checkAvailability()
    }

    class CD {
        -int numberOfTracks
        -String format
    }

    class Tape {
        -String tapeType
        -int duration
    }

    class Invoice {
        -int invoiceId
        -DateTime issuedAt
        -float totalAmount
        +generate()
        +print()
    }

    class ShippingList {
        -int shippingId
        -DateTime shippedAt
        -String destination
        +generate()
        +print()
    }

    class Payment {
        <<abstract>>
        -int paymentId
        -float amount
        -DateTime paidAt
        +process()
    }

    class Cash {
        -float amountReceived
        -float change
    }

    class Check {
        -String checkNumber
        -String bankName
    }

    class BankDraft {
        -String draftNumber
        -String bankName
    }

    class MembershipApplication {
        -int applicationId
        -String applicantName
        -String address
        -DateTime submittedAt
        +submit()
        +approve()
    }

    class OrderProcessingClerk {
        -int clerkId
        -String name
        +verifyMembership()
        +processOrder()
        +verifyAvailability()
        +printInvoice()
        +printShippingList()
    }

    class CollectionDeptClerk {
        -int clerkId
        -String name
        +collectPayment()
        +recordPayment()
    }

    Member <|-- RoyalMember
    Member <|-- RegularMember
    Item <|-- CD
    Item <|-- Tape
    Payment <|-- Cash
    Payment <|-- Check
    Payment <|-- BankDraft

    Member "1" -- "0..*" Order : places
    Order "1" -- "1..*" OrderLine : contains
    OrderLine "1" -- "1" Item : references
    Order "1" -- "1" Invoice : generates
    Order "1" -- "1" ShippingList : generates
    Order "1" -- "1" Payment : paid by
    OrderProcessingClerk "1" -- "0..*" Order : processes
    CollectionDeptClerk "1" -- "0..*" Payment : collects
    MembershipApplication "1" -- "0..1" Member : results in
```

---

## Level 3: Advanced Sequence Diagrams

### 1. Complete Order Processing Flow

```mermaid
sequenceDiagram
    actor M as Royal Member
    participant Sys as System
    participant OPC as Order Processing Clerk
    participant CDC as Collection Dept Clerk

    M->>Sys: Login
    M->>Sys: Place order (2 CDs)
    Sys->>OPC: New order received
    OPC->>Sys: Verify membership
    Sys-->>OPC: Royal member confirmed
    OPC->>Sys: Check availability (CD 1)
    Sys-->>OPC: Available
    OPC->>Sys: Check availability (CD 2)
    Sys-->>OPC: Unavailable
    OPC->>Sys: Order unavailable item (Royal privilege)
    Sys-->>OPC: Backordered
    OPC->>Sys: Apply Royal discount
    OPC->>Sys: Process order
    Sys->>Sys: Generate invoice
    Sys->>Sys: Generate shipping list
    OPC->>Sys: Print invoice
    OPC->>Sys: Print shipping list
    Sys->>M: Send invoice
    M->>Sys: Pay by check
    Sys->>CDC: Payment received
    CDC->>Sys: Record payment
    Sys->>Sys: Save to daily records
    Sys-->>M: Order confirmed
```

### 2. Membership Verification (Success and Failure)

```mermaid
sequenceDiagram
    actor C as Customer
    participant Sys as System
    participant OPC as Order Processing Clerk

    C->>Sys: Attempt to place order

    alt Is a Member
        OPC->>Sys: Verify membership(customerId)
        Sys-->>OPC: Member found (Royal/Regular)
        OPC->>Sys: Proceed with order
        Sys-->>C: Order accepted
    else Not a Member
        OPC->>Sys: Verify membership(customerId)
        Sys-->>OPC: Not a member
        Sys->>C: Send membership application form
        C->>Sys: Submit application
        OPC->>Sys: Approve application
        Sys->>Sys: Create new Regular member
        Sys-->>C: Welcome, you can now place orders
    end
```

### 3. Payment Processing (All Types)

```mermaid
sequenceDiagram
    actor M as Member
    participant Sys as System
    participant CDC as Collection Dept Clerk

    M->>Sys: Select payment method

    alt Cash
        M->>Sys: Submit cash payment
        Sys->>CDC: Cash received
        CDC->>Sys: Record payment
        Sys->>Sys: Calculate change if any
    else Check
        M->>Sys: Submit check(checkNumber, bankName)
        Sys->>CDC: Check received
        CDC->>Sys: Verify check details
        CDC->>Sys: Record payment
    else Bank Draft
        M->>Sys: Submit bank draft(draftNumber, bankName)
        Sys->>CDC: Bank draft received
        CDC->>Sys: Verify draft details
        CDC->>Sys: Record payment
    end

    Sys->>Sys: Save to daily records
    Sys-->>M: Payment confirmed
```

### 4. Item Reordering for Royal Members

```mermaid
sequenceDiagram
    actor R as Royal Member
    participant Sys as System
    participant OPC as Order Processing Clerk

    R->>Sys: Place order for item
    Sys->>OPC: New order
    OPC->>Sys: Check item availability
    Sys-->>OPC: Item unavailable

    alt Royal Member
        OPC->>Sys: Confirm Royal membership
        Sys-->>OPC: Royal status confirmed
        OPC->>Sys: Place backorder for item
        Sys->>Sys: Add to reorder queue
        Sys-->>R: Item backordered, will ship when available
        Sys->>Sys: Item restocked
        Sys->>OPC: Notify item available
        OPC->>Sys: Process backordered item
        Sys->>Sys: Generate shipping list
        Sys-->>R: Item shipped
    else Regular Member
        Sys-->>R: Item unavailable, order cannot be placed
    end
```

