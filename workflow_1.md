# Cookie & Drinks E-commerce Website

## Product Workflow and Functional Requirements

**Document type:** Working product specification / README\
**Project stage:** Planning and MVP definition\
**Proposed stack:** React + Vite + TypeScript, Node.js + Express,
PostgreSQL\
**Architecture:** One frontend application, one backend API (modular
monolith), one relational database

------------------------------------------------------------------------

## 1. Project goal

Build a reliable online shop for cookies and drinks. Visitors can browse
without registering, but must have an account to add products to the cart
and place an order. Customers can track orders, pay by cash on delivery
or online card payment, and review purchased products and their overall
shopping experience after delivery. The catalogue supports both products
available from stock and products that must be made as pre-orders.

The business team uses three operational roles:

-   **Admin:** manages the business and oversees all orders and
    operations.
-   **Worker:** prepares and packages confirmed orders.
-   **Delivery person:** collects prepared orders and delivers them.

The **customer** is the fourth user role. The **backend/system** is not
a human role, but it performs validations, calculations, notifications,
access control, and record keeping.

The system should be designed around complete business workflows---not
merely separate pages.

## 2. Core design principles

1.  **One order lifecycle, role-specific workspaces.** The four roles
    use different routes and permissions, but they operate on shared,
    consistent order data.
2.  **Backend is authoritative.** The server validates prices,
    quantities, stock, delivery eligibility, permissions, order
    transitions, and payment state.
3.  **Separate business states.** Order review, preparation, delivery,
    payment, cash reconciliation, and reviews must not be collapsed into
    one status field.
4.  **Record important events.** Keep timestamps and actor IDs for
    confirmation, cancellation, preparation, pickup, delivery,
    reassignment, payment changes, and moderation.
5.  **Do not silently change an order.** Price changes, unavailable
    products, substitutions, cancellations, and failed delivery attempts
    require an explicit workflow.
6.  **Integrate early.** Frontend mock data must match the agreed API
    contract; the two developers should test the complete journey before
    building many isolated screens.
7.  **Build the reliable core before visual extras.** 3D animation and
    advanced effects come after checkout, order operations, reviews,
    security, and error handling work correctly.

------------------------------------------------------------------------

## 2.1 Confirmed product decisions

These decisions have been made for the current project scope. Treat them
as the working requirements unless the business owner explicitly changes
them.

1. **Mixed stock model:** some products are available from stock and can
   be ordered immediately; other products are made to order as
   pre-orders. Each product must clearly indicate its availability type
   and any expected preparation/availability timing.
2. **Fixed product variants:** flavors are predefined per product. The
   customer chooses from the variants the admin has configured; the
   customer does not freely customize ingredients or create new flavor
   combinations in the MVP.
3. **Stock check:** the system checks stock and product availability
   before accepting/confirming an order. Stock must be rechecked at the
   authoritative backend step to avoid relying on stale browser data.
4. **Account required for cart and ordering:** visitors can browse
   without an account. On first visit, show a registration/sign-in prompt
   that can be dismissed with a “For later” option. Dismissing it allows
   browsing only; adding products to the cart and placing an order
   require authentication. Preserve the intended product/action through
   sign-in where practical.
5. **Delivery fees and service areas:** the admin manually configures
   delivery rules based on addresses/areas. The system saves these rules,
   matches the customer's address to the configured area, calculates the
   delivery fee, and displays it at checkout. If an address cannot be
   matched confidently, do not guess a fee; ask for correction or flag it
   for admin review.
6. **Delivery-first MVP:** customer pickup is out of scope for the first
   version. Focus on delivery operations.
7. **Automatic routine confirmation:** orders may be auto-confirmed when
   all configured checks pass, such as product availability, valid
   required customer information, serviceable address, valid totals, and
   the required payment condition. Flag failed, ambiguous, or suspicious
   checks for admin review. Filled-in information alone does not prove
   that a person or address is genuine.
8. **Payment choice:** the customer chooses cash on delivery or online
   card payment before the order is finalized. Online payment must be
   handled through a suitable payment provider; the backend verifies the
   payment result before marking an order paid.
9. **Payment provider:** not yet selected. Provider support for a
   Morocco-based business, merchant onboarding, MAD or other required
   currency, settlement, fees, and card-payment methods must be checked
   before implementation.
10. **Cancellation:** customers may cancel before preparation starts.
    Once preparation has started, self-service cancellation is disabled;
    the customer must contact the shop for assistance. Refund eligibility
    and processing for paid orders must follow a separate documented
    policy and payment-provider workflow.
11. **Review moderation:** new reviews receive automated checks for
    profanity, spam, and inappropriate content. Reviews that pass the
    checks may be published; suspicious/flagged reviews should be held
    for admin review. Negative sentiment alone is not a reason to hide a
    review.
12. **Review editing/deletion:** customers may edit or delete their
    review within one hour of submission. The backend enforces the
    deadline; after it expires, changes require an admin moderation
    process rather than a client-side-only restriction.
13. **Notifications:** use in-app notifications, SMS, and email for key
    events. A failure in one channel must not undo an order-state change
    or block the other channels. Track attempts and retry safely.
14. **Privacy/retention:** detailed retention rules will be decided later.
    Until then, collect only necessary customer data, restrict staff
    access, and never expose customer addresses/contact information in
    public pages or reviews.

### Important design refinements

- For the MVP, prefer **separate orders** for in-stock and pre-order
  products unless the business explicitly chooses a combined-order
  policy. This avoids unclear delivery promises when items have different
  preparation times.
- Define whether stock is reserved when an order is created or when it
  is confirmed. The recommended approach is an atomic stock reservation
  at successful order creation/confirmation, with release on cancellation
  or payment failure where applicable. The team must implement one
  consistent rule to prevent overselling.
- A customer choosing online payment should be sent to a provider-hosted
  checkout or secure provider component. Never collect or store raw card
  numbers or CVV in this application's database.
- Automated order checks are rules-based screening, not identity
  verification. Any verification requirement should be proportionate
  and explicit.

------------------------------------------------------------------------

## 3. Proposed architecture

``` text
Customer / Admin / Worker / Delivery person
                    |
                    v
          React frontend application
     Public storefront + protected workspaces
                    |
             HTTP / JSON API
                    |
                    v
            Node.js + Express
  Authentication / authorization / business rules
  Products / inventory / orders / delivery / reviews
                    |
                    v
               PostgreSQL
```

Recommended starting choices:

-   **Frontend:** React, Vite, TypeScript, React Router.
-   **Backend:** Node.js and Express.
-   **Database:** PostgreSQL; a relational model suits orders, order
    items, stock, users, payments, and delivery assignments.
-   **ORM:** Prisma is a reasonable beginner-friendly option.
-   **Version control:** GitHub with feature branches and pull requests.
-   **Deployment:** choose the frontend host, backend host, database
    host, and payment provider after checking their current
    availability, cost, and support for Morocco.

Use a **modular monolith**, not microservices. The application can be
organized into modules without the operational burden of running
multiple independent services.

------------------------------------------------------------------------

## 4. Roles, responsibilities, and access boundaries

### 4.1 Customer

**Goal:** discover products, place an order, track it, and optionally
review the order after delivery.

Can: - Browse public products and categories. - View product details,
options, availability, prices, and published reviews. - Manage their
cart. - Submit an order. - View and track only their own orders. -
Request cancellation according to the shop's policy. - Submit eligible
product reviews and a store review after delivery.

Cannot: - Change prices, stock, order statuses, or payment records
directly. - View another customer's private order or address. - Review
products they did not purchase through an eligible delivered order.

### 4.2 Admin

**Goal:** manage the business, coordinate staff, and oversee the entire
order lifecycle. The admin does not normally prepare or deliver orders.

Can: - View all orders and operational queues. - Confirm, cancel, or
handle exceptions for orders under defined rules. - Manage products,
categories, prices, options, and availability. - Manage stock and record
stock corrections. - Assign and reassign preparation tasks and
deliveries. - Manage staff accounts and roles. - Configure delivery
zones/fees and business settings. - Review payment and
cash-reconciliation records. - Moderate reviews under a documented
policy. - View reports and audit history.

Sensitive actions---such as refunds, manual stock corrections, payment
adjustments, and review hiding---should require a reason and be
recorded.

### 4.3 Worker

**Goal:** prepare the correct products, verify the order, package it,
and make it ready for pickup.

Can: - View orders assigned to them or available in an explicitly
configured shared queue. - View the product lines and preparation notes
needed for the task. - Start preparation, update the preparation
checklist, and mark the order ready. - Report unavailable ingredients,
damaged products, delays, or other exceptions. - View their own task
history.

Cannot: - Change prices or the final order total. - Manage staff or
assign delivery people. - Independently cancel orders or approve
substitutions. - Mark a delivery as completed or record a payment unless
explicitly given a separate, narrowly scoped permission.

### 4.4 Delivery person

**Goal:** collect the correct prepared order, deliver it, and record the
actual outcome.

Can: - View deliveries assigned to them. - View necessary delivery
address, contact details, order reference, instructions, and
payment-on-delivery amount. - Confirm pickup. - Record delivery success
or failure and a reason. - Record cash actually received, where
applicable. - View their own delivery history.

Cannot: - Change order contents, prices, or delivery fees. - Browse all
customer orders or accounts. - Mark cash collected before receiving
it. - Mark online payment successful merely because an order was
delivered.

### 4.5 System / backend

The backend must: - Authenticate users and enforce role-based
authorization on every protected operation. - Validate all input and
recalculate totals. - Check stock and delivery eligibility. - Prevent
invalid or conflicting status transitions. - Record timestamps, actor
IDs, and important state changes. - Send or queue notifications. -
Verify online payment provider events server-side. - Ensure duplicate
requests or webhook events do not create duplicate orders or payments. -
Enforce review eligibility and uniqueness rules.

**Security rule:** hiding a button in React is not authorization. The
API must independently check whether the current user can perform the
action on the specific resource.

------------------------------------------------------------------------

## 5. Customer workflow --- from browsing to delivery

### Stage A: Browse the shop

1.  Visitor/customer opens the home page.
2.  The site shows a registration/sign-in prompt with a **For later**
    option. Dismissing it permits browsing but does not grant cart or
    checkout access.
3.  Customer sees shop branding, featured products/categories, and
    relevant availability information.
3.  Customer opens the menu/catalogue.
4.  Customer browses cookies, drinks, boxes, or other categories
    configured by the business.
5.  Customer may search or filter if these features are included in the
    MVP.
6.  Customer opens a product detail page.
7.  Page displays the product name, images, description, price, fixed
    configured flavor variants, availability type, estimated timing for
    pre-orders, and published product reviews.
8.  Customer selects one of the configured variants, if applicable, and
    quantity.
9.  If not signed in, the customer is asked to sign in or register before
    adding the product to the cart. After authentication, return them to
    the intended product/action where practical.
10. Customer adds the product to the cart.

System behavior: - Fetch current product data from the backend. - Do not
treat frontend prices or stock values as authoritative. - Clearly show
whether a product is in stock or available as a pre-order. - Do not offer
free-form customization in the MVP.

### Stage B: Cart

11. Authenticated customer opens the cart.
12. Customer checks products, selected fixed variants, quantities, and
    line totals.
13. Customer changes quantities or removes items.
14. Customer proceeds to checkout.

Unauthenticated visitors cannot add items to or use a persistent cart.
The UI should explain the sign-in requirement clearly.

System behavior: - Update cart calculations. - At checkout, revalidate
current prices and availability. - If an item or price changed, ask the
customer to review the change rather than silently changing the order.

### Stage C: Checkout

15. Customer provides or confirms their name and phone number.
16. Customer provides a delivery address and optional delivery
    instructions.
17. The backend matches the address to admin-configured delivery rules
    and calculates the delivery fee. If the address is ambiguous or
    outside the service area, the customer must correct it or the order
    is sent for admin review.
18. The customer chooses **cash on delivery** or **online card payment**.
19. The customer reviews the subtotal, delivery fee, any supported
    discount, and final total before finalizing.
20. The backend rechecks current prices, selected variants, stock/pre-order
    rules, serviceability, and required customer information.
21. If online card payment is selected, the customer completes payment
    through the selected provider's secure flow. The backend verifies
    the provider result; the browser redirect alone is not proof of
    payment.
22. The system creates/finalizes the order with a unique reference and
    snapshots of product names, selected variants, quantities, unit
    prices, delivery fee, and final total.

System behavior: - The backend is authoritative and recalculates all
amounts. - Prevent duplicate order/payment creation using idempotency
where appropriate. - Save order, preparation, delivery, and payment
states separately. - Never store raw card details. - The exact point of
order creation and stock reservation must be consistent with the
selected payment flow. - For a payment that is pending or fails, show an
accurate pending/failed state and define whether the order expires and
stock is released.

Guest checkout is not part of the current plan: account authentication
is required for cart use and ordering.

### Stage D: Review and confirmation

23. The system runs configured confirmation checks: product/variant
    availability, stock or pre-order rules, valid totals, required
    customer information, delivery serviceability, and the payment
    condition for the selected method.
24. If all checks pass, the order can be automatically confirmed and
    enter the preparation queue.
25. If a check fails, information conflicts, the address is ambiguous,
    or the order is flagged by a configured risk rule, the order enters
    an admin-review queue rather than being silently confirmed.
26. The customer receives the appropriate confirmation, pending-review,
    payment-failure, or cancellation update.
27. Only confirmed orders become eligible for preparation.

The admin retains oversight and can handle exceptions. Automatic
confirmation rules must be explicit, logged, and tested; they must not
be based solely on the fact that form fields are filled in.

### Stage E: Track the order

26. Customer opens order details from the confirmation page,
    notification, or account order history.
27. Customer sees status updates as the order progresses.
28. Customer sees cancellation or delay information when applicable.
29. After delivery is confirmed, customer sees the delivery confirmation
    and a review invitation.

The customer should not be forced to complete a review to use the site.
If the review prompt is closed, they can return through the order
history or notification.

------------------------------------------------------------------------

## 6. Admin workflow --- management and oversight

### Stage A: Login and dashboard

1.  Admin logs in.
2.  Backend verifies the account and role.
3.  Admin sees an overview of:
    -   New orders awaiting review.
    -   Confirmed orders awaiting preparation.
    -   Orders being prepared.
    -   Ready orders awaiting delivery assignment.
    -   Deliveries in progress.
    -   Failed or overdue orders.
    -   Low-stock alerts.
    -   Payment or cash discrepancies.
4.  Admin opens a queue item to investigate.

### Stage B: Review a new order

5.  Admin opens order details.
6.  Admin checks items, quantities, options, customer instructions,
    delivery address, and total.
7.  Admin checks relevant stock and preparation capacity.
8.  Admin checks delivery eligibility and operational constraints.
9.  Admin checks payment state when online payment is required before
    confirmation.
10. Admin confirms the order if it can be fulfilled.
11. Otherwise, admin cancels or holds it according to the defined policy
    and records a reason.
12. Backend records the actor and timestamp and updates the
    customer-visible status.
13. The order enters the preparation queue when confirmed.

### Stage C: Coordinate preparation

14. Admin views confirmed orders awaiting preparation.
15. Admin assigns a worker or leaves the order in a shared queue,
    depending on the chosen policy.
16. Admin monitors preparation progress.
17. If a task is delayed, admin investigates.
18. If a worker reports an unavailable ingredient or product issue,
    admin contacts the customer when necessary and chooses an allowed
    resolution.
19. Admin may reassign the task if the worker is unavailable; assignment
    history is preserved.

### Stage D: Coordinate delivery

20. Admin sees orders marked Ready.
21. Admin checks delivery-person availability and delivery constraints.
22. Admin assigns a delivery person.
23. Backend exposes only the required delivery details to that person.
24. Admin monitors pickup and delivery progress.
25. If the delivery person is unavailable, admin reassigns the task and
    preserves the assignment history.
26. If delivery fails, admin decides the next step: retry, contact the
    customer, return the order, or cancel/close under the business
    policy.
27. Admin investigates cash discrepancies or payment issues.

### Stage E: Manage the shop

28. Admin creates and edits products, categories, descriptions, photos,
    prices, and options.
29. Admin activates/deactivates products and manages availability.
30. Admin updates stock, records restocking, and makes reasoned stock
    corrections.
31. Admin manages staff accounts and role assignments.
32. Admin configures opening hours, delivery zones, fees, and other shop
    settings.
33. Admin reviews orders, payments, cash reconciliation, customer
    complaints, and reports.
34. Admin moderates reviews according to policy, recording the reason
    for hiding a review.
35. Admin can inspect audit history for sensitive actions.

------------------------------------------------------------------------

## 7. Worker workflow --- preparation

### Stage A: Receive work

1.  Worker logs in.
2.  Worker opens **My tasks** or the preparation queue.
3.  Worker sees only assigned orders or tasks eligible for the shared
    queue.
4.  Worker opens a task and checks the order reference, product lines,
    quantities, options, and relevant preparation notes.
5.  Worker verifies that the order has not been cancelled and is
    eligible for preparation.
6.  Worker claims the task if shared-queue claiming is enabled; the
    backend prevents conflicting claims.

### Stage B: Prepare

7.  Worker marks the task `PREPARING`.
8.  Worker collects required ingredients and packaging.
9.  Worker prepares or assembles each product.
10. Worker checks quantities, sizes, flavors, and approved
    customizations.
11. Worker records checklist progress if the checklist feature is
    enabled.
12. If an item cannot be prepared, worker flags an exception and
    provides a reason.
13. Worker does not silently substitute an item or change the order
    total.
14. Admin decides whether to contact the customer, approve a
    substitution, adjust the order, or cancel it.

### Stage C: Verify and hand over

15. Worker checks the completed items against the order.
16. Worker verifies packaging and ensures drinks and all ordered
    products are included.
17. Worker marks preparation `READY`.
18. Backend records the timestamp and notifies the relevant admin/queue.
19. Order becomes eligible for delivery assignment or customer pickup.
20. Worker places the order in the designated handover area.
21. Worker verifies the order reference during handover if the workflow
    requires it.
22. Worker completes the task and it leaves the active preparation
    queue.

The worker must not mark the delivery as completed; preparation
completion and delivery completion are separate events.

------------------------------------------------------------------------

## 8. Delivery-person workflow --- pickup and delivery

### Stage A: Assignment

1.  Delivery person logs in.
2.  Delivery person opens **My deliveries**.
3.  They see only tasks assigned to them.
4.  They open a task and review the order reference, address, necessary
    customer contact details, delivery instructions, and payment method.
5.  For cash on delivery, they can see the amount to collect.
6.  If unable to accept an assignment, they report this through the
    supported workflow; admin can reassign it.

### Stage B: Pickup

7.  Delivery person arrives at the pickup location.
8.  They identify the order using its reference.
9.  They verify that the package is ready and matches the order.
10. They confirm pickup.
11. Backend records the pickup time and changes delivery state to
    `PICKED_UP` or `IN_TRANSIT`.
12. Customer tracking is updated.

The system should not allow pickup confirmation before the order is
ready, unless an explicit exceptional workflow is designed.

### Stage C: Delivery attempt

13. Delivery person follows the address and instructions.
14. They contact the customer when reasonably necessary.
15. They attempt delivery.
16. If the customer is unavailable, the address is wrong, or another
    issue occurs, they record a failed attempt and a reason.
17. Backend notifies admin and keeps the delivery unresolved until the
    next action is decided.
18. Admin determines whether to retry, return, reassign, or close the
    delivery under the shop policy.

### Stage D: Successful delivery and payment collection

19. Delivery person hands the order to the customer or an approved
    recipient.
20. They confirm successful delivery in the app.
21. If cash on delivery applies, they receive the amount and explicitly
    record the amount collected.
22. Backend records the delivery timestamp, actor, and collection event.
23. Customer sees **Delivered**.
24. Review invitation is created and customer notification is triggered.
25. Delivery task moves to history.
26. Cash remains subject to reconciliation; it should not be treated as
    reconciled merely because the driver recorded collection.

------------------------------------------------------------------------

## 9. Order, preparation, delivery, and payment states

These are separate dimensions. Do not combine them into one `status`
field.

### Order review state

-   `PENDING_REVIEW`
-   `CONFIRMED`
-   `CANCELLED`

### Preparation state

-   `NOT_STARTED`
-   `PREPARING`
-   `READY`

### Delivery state

-   `UNASSIGNED`
-   `ASSIGNED`
-   `PICKED_UP` / `IN_TRANSIT`
-   `DELIVERED`
-   `FAILED`
-   `RETURNED` (if the business supports returns)

### Payment state

-   `UNPAID`
-   `PENDING`
-   `PAID`
-   `FAILED`
-   `REFUND_PENDING`
-   `REFUNDED`

### Cash reconciliation state (where applicable)

-   `NOT_APPLICABLE`
-   `AWAITING_COLLECTION`
-   `COLLECTED`
-   `RECONCILED`
-   `DISCREPANCY`

The exact states can be simplified for the MVP, but their meanings must
remain separate.

Example: a confirmed cash-on-delivery order can have preparation
`READY`, delivery `ASSIGNED`, payment `UNPAID`, and cash reconciliation
`AWAITING_COLLECTION`. This is valid.

Do not mark online payment as paid merely because the customer returns
to the site. Verify payment provider confirmation on the backend,
including webhook signatures and duplicate-event handling.

------------------------------------------------------------------------

## 10. Exception workflows

  -----------------------------------------------------------------------
  Problem                 Immediate response      Owner / next step
  ----------------------- ----------------------- -----------------------
  Product unavailable     Do not confirm the      Admin contacts customer
  before confirmation     order as-is             or cancels

  Price or stock changed  Ask customer to review  Customer confirms the
  at checkout             updated cart            revised order

  Ingredient missing      Worker flags exception  Admin decides
  during preparation                              resolution

  Customer requests       Check current stage and Admin handles
  cancellation            cancellation policy     confirmed/in-progress
                                                  orders

  Worker unavailable      Return/reassign task    Admin

  No delivery person      Keep order ready and    Admin assigns when
  available               unassigned; inform      possible
                          admin                   

  Customer cannot be      Record failed attempt   Admin decides
  reached                 and reason              retry/return

  Online payment fails    Do not mark paid        Customer retries or
                                                  admin follows policy

  Cash collected differs  Record actual amount    Admin reconciles
  from expected           and discrepancy         

  Customer complains      Preserve original order Admin investigates and
  about missing/damaged   and complaint           records resolution
  item                                            

  Review contains abusive Flag for moderation     Admin applies review
  content or personal                             policy and records
  information                                     reason
  -----------------------------------------------------------------------

Do not silently overwrite historical information to make an order look
successful. Keep a record of important transitions and corrections.

------------------------------------------------------------------------

## 11. Review feature --- triggered after delivery

The review feature has two separate review types:

1.  **Product review:** belongs to a particular purchased product and
    appears on that product's detail page.
2.  **Store review:** rates the overall shopping experience and can
    appear in the homepage review section.

A delivered order may have no reviews. Reviews are optional and do not
change the order's delivery or payment state.

### 11.1 Trigger and notification

1.  Delivery person confirms successful delivery.
2.  Backend validates that the delivery transition is allowed.
3.  Backend records the delivery timestamp and actor.
4.  Backend creates or enables a review invitation associated with that
    order.
5.  System queues notifications for **in-app, SMS, and email**. Each
    channel has its own delivery/attempt status so a temporary failure
    in one channel does not block the others.
6.  Each notification links to the review page for that specific order.
7.  The delivery confirmation screen also shows a prominent **Review
    your order** action.
8.  Do not forcibly redirect the customer or block them from leaving.
9.  If the customer dismisses the prompt, they can return to it from
    their order details or notification.

Notifications should be idempotent: repeated status requests must not
create duplicate invitations or repeatedly send the same delivery
notification. Store channel-specific delivery/attempt state and retry
transient failures safely. The order's delivered status must not depend
on whether an SMS or email provider is temporarily unavailable.

### 11.2 Review page

The page loads only the products belonging to the eligible delivered
order.

For each distinct product in the order: 1. Show product name and
thumbnail. 2. Show a required 1--5 star rating control. 3. Provide an
optional written comment. 4. Offer one review per product per order by
default, even if the quantity is greater than one. 5. If the product
already has a review for this order, show the saved review or an edit
option according to the chosen policy.

Then show a separate optional store review: 1. Required 1--5 star rating
if the customer chooses to submit a store review. 2. Optional written
comment about the overall experience, such as ordering, packaging,
service, and delivery.

Submission: 1. Customer submits the product review(s) and optional store
review. 2. Backend verifies the order belongs to that customer (or
verifies secure guest-order access). 3. Backend verifies that delivery
is confirmed. 4. Backend verifies that every reviewed product appears in
that order. 5. Backend validates ratings and text length. 6. Backend
enforces review uniqueness and saves the submission. 7. Backend returns
a clear success or validation message. 8. Frontend shows a thank-you
state and allows the customer to see what they submitted.

A customer may submit product reviews without a store review, or vice
versa. Do not require every optional review to be completed in one
session.

### 11.3 Product-page review display

Each product page should have a review section containing: - Average
rating calculated from all eligible published reviews. - Total number of
eligible published reviews. - A selection of **4--6 distinct reviews**
shown at a time. - Rating, comment, display name or approved pseudonym,
and date where appropriate. - A **See all reviews** link or expandable
section if the full list is supported.

Selection logic: 1. Fetch eligible published reviews for that product.
2. Randomly select up to six distinct reviews. 3. Keep the selection
stable for the current page view; do not rerandomize on every React
re-render. 4. A new page visit or controlled refresh may produce a
different selection. 5. If there are fewer than four reviews, show all
available reviews. 6. If there are no reviews, show a friendly empty
state such as "Be the first to review this product."

The displayed sample must not determine the average rating. The rating
and count must be calculated across all eligible published reviews.
Never fabricate reviews, repeat the same review to fill space, or
display hidden/pending reviews publicly.

### 11.4 Homepage store-review display

Near the bottom of the homepage, before the footer, show a section such
as **"Loved by our customers"**.

It should display: - Overall store rating calculated from all eligible
published store reviews. - Total count of eligible store reviews. -
**Three distinct store reviews** selected randomly. - Each review's
rating, short comment, display name or approved pseudonym, and date if
desired. - An optional link to all store reviews.

Selection logic: 1. Retrieve eligible published store reviews. 2.
Randomly select up to three distinct reviews. 3. Keep the selection
stable for the page view. 4. A later visit or controlled refresh can
show a different selection. 5. If fewer than three reviews exist, show
only those available. 6. If none exist, hide the section or show a
neutral invitation; do not use fake testimonials.

The homepage's overall rating must be calculated from all eligible
published store reviews, not just the three displayed.

### 11.5 Review moderation and trust

Recommended review states: - `PENDING` (if moderation before publication
is enabled) - `PUBLISHED` - `HIDDEN`

MVP policy: run automated checks for profanity, spam, and inappropriate
content before publication. Reviews that pass may be published
automatically; reviews flagged by the checks are held for admin review.
Automated text checks can make mistakes, so provide an admin path to
review flagged content and correct false positives.

Do not hide a review solely because it is negative. Moderation should
follow a clear policy covering spam, abuse, personal information,
irrelevant content, or other defined violations. Record the moderation
decision, reason, actor, and time. Enforce the one-hour customer
edit/delete window on the backend, not only in the frontend.

### 11.6 Review data model

A relational design can use separate `ProductReview` and `StoreReview`
tables.

**ProductReview** - `id` - `customer_id` (nullable or adapted for
securely identified guest orders) - `order_id` - `product_id` - `rating`
(integer 1--5) - `comment` (optional) - `status` - `created_at` -
`updated_at` - optional moderation fields such as `moderated_by`,
`moderation_reason`, `moderated_at`

Enforce a unique constraint on `(order_id, product_id)` for one product
review per order-product pair.

**StoreReview** - `id` - `customer_id` (nullable or adapted for securely
identified guest orders) - `order_id` - `rating` (integer 1--5) -
`comment` (optional) - `status` - `created_at` - `updated_at` - optional
moderation fields

Enforce one store review per qualifying order, for example with a unique
constraint on `order_id`.

A review invitation/completion state can be stored on an order-related
review-invitation record or derived from review records. Avoid
maintaining duplicate states that can disagree without a clear reason.

### 11.7 Review API contract to plan

Example endpoints: - `GET /api/orders/:orderId/review` --- return the
eligible order and products for the review form. -
`POST /api/orders/:orderId/reviews` --- submit product reviews and an
optional store review. - `GET /api/products/:productId/reviews` ---
return rating summary and a selected set of published reviews. -
`GET /api/store-reviews` --- return the store rating summary and a
selected set of published store reviews. - Admin-only endpoints for
moderation and review status changes.

These are proposed routes, not implemented endpoints. The backend must
verify authorization, delivery status, product eligibility, rating
range, uniqueness, and visibility. Do not trust client-supplied customer
IDs, order ownership, review counts, or averages.

------------------------------------------------------------------------

## 12. Suggested sitemap

### Public/customer routes

``` text
/
├── /menu
├── /products/:productId
├── /cart
├── /checkout
├── /order-confirmation/:orderId
├── /orders/:orderId
├── /orders/:orderId/review
├── /account/orders
├── /store-reviews              (optional)
├── /login
└── /register                   (if accounts are supported)
```

### Admin routes

``` text
/admin
├── /orders
├── /orders/:orderId
├── /products
├── /inventory
├── /preparation
├── /deliveries
├── /staff
├── /payments
├── /reviews
├── /reports
└── /settings
```

### Worker routes

``` text
/staff
├── /tasks
├── /tasks/:taskId
└── /history
```

### Delivery routes

``` text
/delivery
├── /tasks
├── /tasks/:deliveryId
└── /history
```

The paths are examples. One React app can render different layouts and
protected routes. They do not need to be four separately deployed
applications.

------------------------------------------------------------------------

## 13. Proposed core data entities

Design the exact schema only after confirming the business rules, but
the following entities are likely needed:

-   `User` --- identity, credentials/identity-provider data, active
    state.
-   `Role` or a constrained role field --- customer, admin, worker,
    delivery.
-   `Product` --- catalogue details, active state, pricing.
-   `Category` --- product grouping.
-   `ProductOption` or variant tables --- sizes, flavors,
    customizations, if supported.
-   `Inventory` / `StockMovement` --- stock and auditable adjustments,
    if inventory is tracked.
-   `Order` --- customer/contact snapshot, address snapshot, totals,
    timestamps, order-review state.
-   `OrderItem` --- product reference plus snapshot of product name,
    options, unit price, quantity, and line total.
-   `PreparationTask` or preparation fields on the order --- worker
    assignment and preparation history.
-   `Delivery` / `DeliveryAssignment` --- delivery status, assigned
    person, pickup/delivery timestamps, failure reason.
-   `Payment` --- method, provider reference, amount, payment state;
    never raw card details.
-   `CashCollection` / reconciliation record --- actual cash collected
    and reconciliation details for COD.
-   `ProductReview` --- product-specific review linked to order and
    product.
-   `StoreReview` --- overall store review linked to qualifying order.
-   `Notification` or notification/outbox records --- recipient, type,
    related order, delivery/attempt state.
-   `AuditEvent` --- sensitive state changes and administrative actions.

Do not create every table mechanically. Some can be simplified for the
MVP, but preserve the separation of order items, payments, delivery
assignments, and reviews.

**Order-item snapshots matter:** if the product name or price changes
later, historical orders and reviews must still refer to what the
customer actually bought.

------------------------------------------------------------------------

## 14. API areas to agree on before parallel development

The frontend lead and backend lead should agree on request/response
shapes and error formats for:

-   Authentication and current user.
-   Public product catalogue and product details.
-   Cart validation and checkout/order creation.
-   Customer order history and order tracking.
-   Admin order review and status transitions.
-   Product/category management.
-   Inventory changes.
-   Preparation queue and task updates.
-   Delivery assignment, pickup, and outcome.
-   Payment initiation, provider confirmation, and refund handling.
-   Review eligibility, submission, display, and moderation.
-   Notifications.

For each endpoint, document: - HTTP method and route. - Required role. -
Request body and response shape. - Validation errors. - Allowed state
transitions. - Whether the operation is idempotent. - Data the caller is
permitted to see or change.

Frontend mock data should match this contract. Do not let the two
developers independently invent different shapes for the same order.

------------------------------------------------------------------------

## 15. Two-person team workflow

### Shared responsibilities

Both developers should: - Agree on requirements and edge cases. - Design
the order lifecycle and database together. - Review the API contract. -
Agree on naming conventions and status values. - Integrate regularly and
review pull requests. - Test the complete journey and authorization
rules.

### Developer A --- frontend lead

Owns: - React layouts, routes, and reusable UI components. - Customer
storefront, product pages, cart, and checkout. - Admin, worker, and
delivery workspaces. - Order tracking and review submission/display. -
Loading, empty, error, and success states. - API client integration and
frontend tests.

### Developer B --- backend lead

Owns: - Node.js/Express setup and API modules. - Database schema,
migrations, and ORM setup. - Authentication and authorization. -
Product, order, stock, preparation, delivery, payment, review, and
notification rules. - Validation, error handling, audit records, and
backend tests. - Payment provider integration after provider suitability
is verified.

These are ownership areas, not silos. Both developers should understand
the complete order flow and review each other's work.

------------------------------------------------------------------------

## 16. Recommended implementation milestones

### Milestone 1 --- Requirements and design

-   Confirm stock vs made-to-order rules.
-   Decide guest checkout/account requirements.
-   Define delivery zones, fees, and pickup policy.
-   Define order confirmation and cancellation rules.
-   Define COD collection/reconciliation policy.
-   Agree on review eligibility and moderation policy.
-   Draft sitemap, data model, status transitions, and API contract.

### Milestone 2 --- Foundation

-   Create repository and frontend/backend projects.
-   Configure environment variables and `.gitignore`.
-   Configure database and migrations.
-   Implement health check, error handling, and authentication
    foundations.
-   Agree on API response/error conventions.

### Milestone 3 --- First end-to-end order

-   Display products.
-   Add products to cart.
-   Validate checkout.
-   Create an order and order-item snapshots.
-   Show the order in the admin queue.
-   Test invalid products, changed prices, and insufficient stock.

### Milestone 4 --- Admin and preparation

-   Confirm/cancel orders.
-   Manage products and stock.
-   Add worker queue, task claiming/assignment, preparation progress,
    and ready state.
-   Add audit history and exception handling.

### Milestone 5 --- Delivery

-   Assign delivery person.
-   Add pickup, in-transit, delivered, and failed states.
-   Add COD collection records and reconciliation.
-   Add customer order tracking and delivery notifications.

### Milestone 6 --- Reviews

-   Trigger a review invitation on successful delivery.
-   Notify the customer and make the review form accessible later.
-   Save product reviews and optional store reviews.
-   Display 4--6 product reviews per product page and three store
    reviews on the homepage, with graceful empty states.
-   Add rating summaries, uniqueness constraints, and moderation policy.

### Milestone 7 --- Online payments and hardening

Before integrating a provider, verify that it supports a Morocco-based
merchant, the required settlement currency, onboarding, refunds, and
the card methods the shop intends to accept. Do not select a provider
based on customer Visa/Mastercard ownership alone: merchant-country
support and settlement are decisive.

-   Select a provider after checking availability for a Moroccan
    business, supported currency, fees, settlement, and onboarding.
-   Use the provider's secure checkout components/hosted flow.
-   Verify payment results server-side.
-   Test duplicate webhooks, failed payments, cancellation, refunds, and
    reconciliation.
-   Test role permissions, data access, and common security failures.

### Milestone 8 --- Release and polish

-   Deploy frontend, backend, and database.
-   Configure secrets and production environment variables.
-   Test on phone and desktop.
-   Add monitoring, backups, and error reporting.
-   Improve design, accessibility, performance, and purposeful
    animation/3D.

------------------------------------------------------------------------

## 17. Acceptance criteria for the first complete release

The core workflow is not complete until the team can demonstrate all of
the following without manually editing database rows:

1.  Customer browses products and places a valid order.
2.  Backend recalculates the total and saves an order with item
    snapshots.
3.  Admin sees and confirms the order.
4.  Worker prepares it and marks it ready.
5.  Admin assigns a delivery person.
6.  Delivery person confirms pickup and delivery.
7.  Customer sees updated order progress.
8.  COD collection or online payment state is accurately represented.
9.  Successful delivery triggers one review invitation/notification.
10. Customer can return to the review page if they do not submit
    immediately.
11. Customer can submit a product review for each eligible purchased
    product and an optional store review.
12. Product pages display up to six distinct published product reviews;
    homepage displays up to three distinct published store reviews.
13. Review averages/counts use all eligible published reviews, not just
    the visible sample.
14. Empty review states work without fabricated content.
15. Duplicate review submissions are prevented according to the policy.
16. A customer cannot review a product they did not purchase in the
    eligible order.
17. Workers and delivery people cannot access or change resources
    outside their permissions.
18. Failed preparation, cancellation, failed delivery, payment failure,
    and cash discrepancies have defined paths.
19. Important actions have actor/timestamp records.
20. Refreshing a page or retrying a request does not create duplicate
    orders, notifications, or payments.
21. Visitors can browse and dismiss the sign-in prompt, but cannot add
    products to the cart or order without signing in.
22. Fixed configured flavors are selectable; unsupported free-form
    customization is not offered.
23. Stock products and pre-order products are clearly distinguished,
    and pre-order timing is visible.
24. Delivery fees come from admin-configured address/area rules; an
    ambiguous or unsupported address is not assigned a guessed fee.
25. Routine orders can be automatically confirmed only after all
    configured checks pass; exceptions are routed to admin review.
26. Online payments are marked paid only after server-side provider
    verification.
27. Customer self-service review edit/delete is available for one hour
    after submission and is enforced by the backend.
28. Delivery review invitations queue in-app, SMS, and email messages
    without duplicate sends or one-channel failures blocking the order.

------------------------------------------------------------------------

## 18. Decisions still to confirm

The following details remain open. Resolve them before the related
feature is implemented; avoid silently inventing a policy in code.

- **Stock reservation timing:** reserve stock when the order is created,
  when payment succeeds, or when the order is confirmed? Choose a single
  consistent rule for each payment path and prevent overselling.
- **Pre-order promise:** what expected preparation/availability time
  should be shown, and how should delays be communicated? For the MVP,
  should pre-order products be ordered separately from in-stock items?
  Recommended default: separate orders.
- **Automatic confirmation rules:** which checks are mandatory, what
  constitutes a risk flag, and which exceptions require admin review?
- **Address matching:** how will customers enter addresses, how will the
  system map them to admin-configured areas, and when should a human
  resolve an ambiguous address?
- **Payment provider:** which provider supports the business's country,
  merchant onboarding, currency, settlement, fees, refunds, and desired
  card methods? Verify current provider requirements before committing.
- **Payment failure/timeout:** how long does a pending online payment
  remain valid, when is an order expired, and when is reserved stock
  released?
- **Refund procedure:** who authorizes a refund, which cancellation or
  failure cases qualify, and how is provider refund status tracked?
- **Delivery operations:** delivery hours, service boundaries, expected
  delivery times, delivery assignment rules, failed-delivery retry
  limits, and return-to-shop procedure.
- **Review edge cases:** can a review be edited/deleted after one hour
  through an admin request, and what appeal process exists if automated
  moderation flags a legitimate review?
- **Notification provider and cost:** choose SMS and email providers,
  configure templates, and decide retry limits and fallback behavior.
- **Privacy/retention:** define retention for customer accounts, phone
  numbers, addresses, order records, delivery details, and reviews.
- **Order cancellation:** confirm the exact cutoff rule using the
  preparation state. Recommended rule: customer self-cancellation is
  available only before preparation starts; afterward, the customer
  contacts the shop.

Decisions already made and recorded in Section 2.1 should not be treated
as unresolved unless the business owner explicitly reopens them.

------------------------------------------------------------------------

## 19. Final recommendation

Build the system around this dependable path:

**Browse → Sign in/register → Cart → Checkout and payment choice →
automated checks → automatic confirmation or admin exception review →
Preparation → Delivery assignment → Pickup → Delivery confirmation →
review invitation → Product and store reviews.**

Keep the order, preparation, delivery, payment, cash reconciliation, and
review states separate. Enforce permissions and business rules on the
backend. Design all four role workflows before coding, but implement and
test the application incrementally.

The first milestone is not a perfect visual design. It is one complete,
secure order journey that works from the customer's browser through the
database and back to the customer. Once that works, expand the
operational features and polish the interface.
