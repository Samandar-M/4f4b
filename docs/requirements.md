# 4f4b V1 Requirements

Status: consolidated development baseline, 18 October 2026. Prepared from accepted project decisions. Open implementation choices are listed in section 16; they are not silently treated as settled requirements.

## 1 Purpose and scope

Make managing inventory, sales, and store personnel simple, while allowing each business to configure the system around its own products, pricing rules, and workflows.

4f4b is one configurable retail management and point of sale application. The plumbing business is its first pilot configuration. Its product names, brands, categories, and stock are business data, rather than rules embedded in the application.

V1 provides a responsive web application for desktop, touchscreen, tablet, and phone browsers. Each business operates one store in V1. The same application supports two delivery models:

- Managed hosting: independent businesses, including the pilot store, can subscribe to a cloud service operated by the 4f4b provider. Each business has its own owner, staff, settings, and private business records.
- Self-hosting: a business can run the application on its chosen cloud infrastructure or a local server. This uses the same codebase with deployment configuration, rather than requiring a separate product or source-code fork.

Store isolation is required from the initial persistent data design. Every account and operation is scoped to the appropriate store; one business cannot access another business's records. Provider operations and subscription administration are distinct from a store's owner and staff permissions. Automated subscription billing has not yet been specified.

V1 requires a live connection to its application server to finalize sales and other transactions. A remotely hosted server requires network access to that server. A local server may remain reachable over the store's local network during an internet outage, allowing ordinary server-connected operations to continue. Offline transaction capture on an isolated client, local inventory snapshots used for checkout, and later synchronization are future capabilities. Such offline support can be implemented in a web or native application; a native application is not a prerequisite.

| Classification | Meaning | Examples |
| --- | --- | --- |
| Core | Always enforced by the application | Permission checks, preserved transaction history, consistent stock and financial records |
| Configurable | A store chooses supported values or policies | Product units, price thresholds, credit limits, staff permissions |
| Optional use | Available in V1, but not required for every product or store | Barcodes, images, bulk pricing, customer credit, measured portions, two-factor authentication |
| Pilot example | Demonstrates a general capability | A 25 mm product stocked in several measured lengths |

Optional use does not mean postponed development. A supported option may be left unused by a business.

Native mobile applications, offline checkout and synchronization, multiple branches or warehouses within one business, supplier ordering, automatic currency conversion, machine learning forecasts, and approximate search suggestions are outside V1. Full business accounting, including payroll, rent, tax reporting, and balance sheets, is also outside V1.

## 2 Core record guarantees

- Completed sales, payments, refunds, revisions, and inventory events retain their original records. Corrections append linked entries; voiding does not erase a completed sale.
- Product changes do not reinterpret historical transactions. Sale records preserve product identification, units, quantities, standard and actual prices, acquisition costs, currency, responsible account, approvals, and time as applicable.
- Protected actions and sensitive data require permission checks, including through the application programming interface. Hiding a button is insufficient.
- Stock changes and their history entries succeed together. Sales, refunds, revisions, and their required inventory and financial effects also succeed together, or none take effect.
- Confirming an operation rechecks current stock, refundable amounts, and applicable approvals. Simultaneous users and repeated confirmation must not create duplicate transactions or excessive stock deductions or refunds.
- Money and measured quantities use exact decimal calculations with defined rounding. Available quantities cannot become negative through ordinary operations.
- Historical references survive product archiving and staff deactivation. Deactivation prevents future access without removing the account from earlier records.
- These guarantees preserve record integrity. They do not turn configurable business warning thresholds into universal prohibitions.

## 3 Product catalog and categories

### Product creation and maintenance

An authorized user can create and maintain products. A product includes:

- Name and category.
- Unit of measurement, allowed sale increment, and whether it may be divided.
- Initial available stock, including portion sizes and counts when applicable.
- Acquisition cost, standard selling price, and seller price threshold, expressed on an identified unit basis.
- Low-stock threshold in the product's unit of measurement.
- Optional bulk-price rules, product image, Stock Keeping Unit (SKU), and barcode.

Every product has a stable internal identifier even when it has no SKU or barcode. An entered lookup identifier must resolve unambiguously within the store. Opening stock is recorded in inventory history.

Categories support creation, renaming, nesting, product assignment, and removal. Removing a category must account for its products and child categories without losing those records. The exact reassignment interaction can be decided during catalog implementation.

Archiving hides a product from ordinary selling and search while preserving historical references. A product may also be temporarily disabled from sale without changing its physical quantity. Neither action silently deletes stock.

### Product lookup

- Sellers can use product-name search, SKU search, barcode input or scanning, and nested category navigation.
- Name and category search is case-insensitive and matches consecutive characters.
- Ranking prioritizes exact SKU or barcode matches, exact product names, product-name prefixes, then partial product-name or category matches.
- Category results look distinct from product results. Selecting one opens that category for further browsing.
- Product results show the name, category path, unit, standard selling price, available quantity, and optional image. Product descriptions must distinguish relevant variants such as size or brand.
- A recognized barcode adds the available product to the cart. Portion selection and quantity confirmation still apply when required. An unknown barcode displays a clear product-not-found message.
- SKU, barcode, and image remain optional. Products without them work through search and categories.
- Out-of-stock products stay visible but cannot be added to a cart. Disabled or archived products cannot be sold.
- Empty search results display a clear message. Approximate spelling suggestions are postponed.

## 4 Units and measured portions

Ordinary counted products use whole quantities. Measured products can use fractional quantities in their configured increments.

A measured product may be stocked in multiple ready-to-sell sizes. 4f4b keeps a count for each size and separately tracks remainders by size. Identical pieces share a count; V1 does not require an individual serial number for every piece.

Total available quantity is calculated from the available sizes and their counts. Faulty stock is excluded. Ready-to-sell portions and remainders remain distinguishable even when their sizes are equal.

During a sale, the seller specifies the portions or amounts the customer will receive. 4f4b suggests an exact portion or a suitable existing remainder, and the seller confirms the source used. The suggestion is advisory. A non-divisible portion must be sold whole.

Completion reduces the selected source count, records the quantity sold, and adds any usable remainder. The related movements share a sale or operation reference in inventory history. Multiple supplied portions remain individually visible beside their combined total.

The system checks the available source sizes as well as the total quantity. Several smaller portions cannot silently satisfy a request for one continuous portion. Source selection cannot mix incompatible products or variants.

The selected source and resulting remainder must remain traceable when a sale is returned, revised, or voided. A financial correction cannot physically recombine divided stock. Unusable remainder, division loss, and cancelled sales after physical division require an explicit inventory treatment; see section 16.

## 5 Inventory management and history

Users with inventory-management permission can find products through the same catalog tools used in sales. They can record restocking, correct counts, move quantities between available and faulty stock, disable products, and archive them as permitted.

Manual stock adjustments require a predefined reason or a written explanation. Examples include restock, physical-count correction, damage, and return to available stock. Available and faulty quantities are tracked separately; faulty quantities are excluded from sale and appear in an inspection list.

Customer returns follow the linked return and refund workflow in section 9. They must not also be entered as a second unrelated stock addition.

Every stock change creates an immutable history entry containing:

- Stable product identification and the affected portion group when applicable.
- Event type and affected stock condition, such as available or faulty.
- Previous quantity, quantity change, and resulting quantity.
- Responsible user, reason, optional note, and date and time.
- Reference to the sale, return, refund, revision, division, or restock when applicable.

Opening stock, sales, returns, and corrections all generate entries. Changing product availability or archiving is also audited, even when no quantity changes.

Entries cannot be edited or deleted through 4f4b. Errors require a linked corrective or reversing entry that reflects the actual stock position. History supports pagination and filters for product, event type, user, and date. Linked records provide detail without copying entire sales into every inventory event.

## 6 Prices and approval

Each product has three identified values on the same unit basis:

- **Acquisition cost:** the product cost used in profitability calculations. Visible only with permission.
- **Standard selling price:** the ordinary default unit price.
- **Seller price threshold:** the lowest unit price a seller can use without approval.

Optional bulk rules pair a minimum purchase quantity with a unit price. A qualifying rule applies automatically and is displayed in the cart. Sellers can negotiate individual line prices; V1 does not provide an independent cart-total override.

Prices below the seller threshold require an authorized approval. Prices below acquisition cost also trigger a loss warning. Bulk prices remain subject to the same approval checks; automatic pricing does not silently bypass them.

The approver can inspect the cart, quantities, standard and proposed prices, thresholds, acquisition costs, reduction in expected profit, and actual loss where applicable. The seller sees the decision and necessary warnings; submitting a request does not itself grant access to restricted costs or profit figures.

A pending or denied request prevents completion at the requested price. The seller can correct the price or submit a new request. Changing cart contents, quantities, or prices invalidates an approval covering the earlier cart. Completion records the approving account and approved values.

The owner can make business exceptions through authorized actions. A threshold is an approval boundary, not an absolute ban on that price.

## 7 Sale and payment workflow

1. An authenticated seller starts a sale and locates products through search, categories, or barcode.
2. The seller adds products, specifies quantities and measured sources where applicable, and can adjust or remove draft cart lines.
3. 4f4b checks quantities and source availability, applies standard or qualifying bulk prices, and displays unit prices, quantities, line totals, and the sale total.
4. Negotiated prices follow section 6. Required approvals must be valid before completion.
5. The seller records any payment received and its method, such as cash or an electronic payment. Electronic payment recording does not imply that V1 processes the payment itself.
6. Payment status is calculated from the amount paid: fully paid, partially paid, or unpaid/deferred. Partial and deferred sales require a linked customer profile and credit permission.
7. Confirmation records the sale, its historical prices and costs, portions, payments, seller, approvals, and completion time. It creates inventory movements and financial and customer-ledger entries together.

Recording a payment means recording money actually received. An outstanding amount remains visible until resolved. Later debt payments belong to the customer ledger and must not create another sale. Supported payment methods and the treatment of change or overpayments will be finalized before checkout implementation.

When the configured server cannot be reached, V1 displays the connection problem and cannot confirm a new sale on the client alone. Internet availability is not the deciding condition when the server is on the local network. If connectivity fails after a confirmation request was sent, the sale may already have completed on the server. The interface shows an unconfirmed outcome until it can check that same operation's status. Reconnecting or retrying must retrieve or complete that operation without creating a second sale.

## 8 Customer ledger and credit

A permitted seller can create a customer profile or find an existing one during checkout. Name and a reliable contact method are required for credit; an address is optional. A customer contact field does not imply phone or email verification.

The sale displays the amount paid now, new unpaid amount, existing customer balance, and resulting balance. The credit check is:

`Resulting balance = existing outstanding balance + new unpaid amount`

The store may configure a credit limit. Exceeding it triggers a warning showing the excess. A user with permission to exceed it can proceed, with their identity, time, and resulting balance recorded. A customer can make another credit purchase while older debt remains, subject to these same checks.

Sales remain visibly outstanding while their own balance is positive. A user with payment-recording permission can record a later payment, method, time, and responsible account. The payment is allocated to the sale or sales it settles; the allocation method is an open implementation decision.

The ledger shows credit sales, payments, refund credits, corrections, and the balance after each entry. It shows both the customer's total debt and each sale's remaining balance. Earlier entries are preserved.

Authorized managing users receive a notification for a completed credit sale. Outstanding-balance summaries can be scheduled daily, weekly, or monthly and combined with the reporting notification when due together.

Revenue and product profit arise from the sale; later debt collection records money received without creating additional revenue.

## 9 Returns and refunds

A user with refund permission finds the original completed sale by ID, date, customer, or product lookup. The sale shows the effective sold quantities and prices after valid revisions, earlier returns, and the remaining quantities eligible for refund. Earlier refund records retain their own values.

The store sets a refund window measured from the original completion time. A user with refund-override permission can approve an exception with a recorded reason. A sale revision does not restart the window.

A full or partial return identifies the products, quantities, and condition:

- Sellable accepted goods increase available stock.
- Accepted faulty or manufacturer-defective goods increase faulty stock and notify authorized inventory users.
- The current default rejects customer-damaged or otherwise ineligible goods without changing stock or finances. Rejected attempts record the user, time, items, quantities, and reason. Broader policy exceptions remain a store-policy design decision.

Physical returns restore the portions actually received. A returned fractional portion is not automatically counted as an intact factory portion.

At confirmation, the system checks:

`Remaining refundable quantity = effective quantity sold - quantities already accepted as returned`

The refund value uses the applicable actual selling price, including negotiated or bulk pricing and valid sale corrections. It cannot duplicate an earlier quantity or monetary refund.

For a paid sale, record the money returned and refund method. For a sale with debt, first reduce the outstanding balance on that sale; only the excess refund value becomes money owed back. Money returned cannot exceed valid payments received for that sale minus money already returned. Debt on an unrelated sale is not silently reduced.

Each accepted return/refund creates one linked operation with the physical return and financial outcomes recorded distinctly. The record contains products, quantities, portions, price basis, condition, reason, debt reduction, money returned or still owed back, method where applicable, user, approver, and time.

The original sale is preserved. Refunds appear independently and through the sale, inventory history, and financial history. Inventory, refund limits, debt, and required financial entries commit together. No actual money return is implied by changing stock alone.

## 10 Sales inspection revisions and voids

Each completed sale has a permanent ID. The searchable sales list shows date, seller, total, payment status, outstanding balance, and indicators for refunds, revisions, and voids. Detail views link the original and current corrected sale to payments, ledger entries, approvals, refunds, and inventory events, subject to permissions.

A user with sale-revision permission can correct the product, quantity, selling price, amount recorded as paid, or payment method. The revision records old and new values, user, reason, approval where required, and time. It appends a linked correction; it does not erase the original record.

Revisions recheck stock, source portions, price approvals, payments, customer balances, and existing refunds. They cannot reduce sold quantities below returned quantities, or invalidate already completed refunds. Product or price changes must preserve the references and values used by existing refunds. An incompatible change is rejected with an explanation until its linked records are resolved through explicit corrections.

Correcting a payment amount or method creates a payment correction within the revision operation. It must distinguish an earlier recording error from an actual new payment or money returned. Any newly owed amount is shown and reconciled through an authorized payment, debt, or refund entry.

Inventory corrections reflect what physically happened. A return belongs to section 9. A revision must not both reverse stock and create a second return for the same goods, or reconstruct a whole portion from already divided material. Required movements are recorded explicitly.

A user with void permission can void a sale entered in error. The sale remains searchable, marked voided, with user, time, reason, and linked reversing entries. Payments, prior refunds, debts, and actual stock must be reconciled before completion. Voiding does not assert that money or goods physically returned without recording that event.

The current sale view reflects accepted corrections. The immutable timeline retains the original sale, revisions, payments, refunds, and void. Related changes succeed together or leave the transaction unchanged. Reports show dated adjustments without counting the original sale or an adjustment twice.

## 11 Low stock and shopping list

Authorized users configure per-product low-stock thresholds in the product's unit. At or below the threshold, available stock is marked low in inventory and sales. Faulty stock is excluded; a measured product's total available quantity is derived from its portions.

The owner is notified once when a product enters the low-stock state. Further reductions within that state do not repeatedly notify. The warning clears above the threshold; a later crossing can produce a new alert. Stock with enough total quantity may still lack a particular requested portion size.

The owner can add products to the shopping list manually or enable automatic addition on entering low stock. Automatic additions do not duplicate active entries. A user's removal is not immediately undone by refreshing the same low-stock state.

Permitted users can add entries, set or change desired purchase quantities, cross out completed entries, and delete entries. The list is a purchase plan: restocking never changes its quantities, crosses out its entries, or deletes them. Users maintain it explicitly. Editing the list never changes inventory; receiving goods is a separate inventory operation.

## 12 Warnings and notifications

Business warnings and approval requests must distinguish an advisory message from an action requiring permission or approval.

| Event | Recipient or display | Result |
| --- | --- | --- |
| Product enters low stock | Inventory and sales display; owner notification | Persistent stock warning; optional shopping-list addition |
| Proposed price below seller threshold | Seller and authorized approver | Sale requires valid approval or a corrected price |
| Price below acquisition cost | Seller warning; authorized approver sees restricted detail | Additional loss warning; ordinary permission and approval checks still apply |
| Faulty stock confirmed | Authorized inventory users and owner | Faulty inspection list and linked history |
| Credit sale completed | Owner or authorized managing users | Notification and ledger entry |
| Proposed balance exceeds credit limit | Seller or authorized user | Warning; permitted override is recorded |
| Scheduled report due | Owner | Daily, weekly, and monthly summaries combined when due together |

Ordinary completed sales appear in live history without individual owner notifications. Report delivery and warning detail must honor sensitive-data permissions. Notification channels beyond the in-application view are not assumed; see section 16.

## 13 Sales and profitability reports

The owner can inspect sales and related records as operations complete. Daily summaries are generated at the configured closing time; weekly and monthly summaries at their respective period ends. The store configures its time zone and week-ending day. Summaries due together are delivered in one notification.

Each summary includes:

- Completed sale count, with voided sales separately identified.
- Cost of goods sold from the recorded acquisition costs.
- Net sales revenue from actual prices, accounting for refunds, voids, and revisions.
- Gross product profit from net sales revenue less the associated cost of goods sold.
- Money received in the period, including payments on older sales.
- Customers with positive outstanding balances and their combined debt at period end.
- Completed refund count and value, distinguishing money returned from debt reductions.
- Newly confirmed faulty quantities, grouped by product and unit.

Revenue, money received, debt reductions, and money returned remain distinguishable. Debt collection is not a new sale. Restocking expenditure is not automatically the cost of goods sold for that period. Gross product profit excludes operating expenses.

Refunds, voids, and revisions appear as dated adjustments linked to their source records. Historical prices and cost snapshots remain preserved. Faulty stock must not be treated as sellable inventory merely to reverse a profit figure. Cost valuation, faulty-stock accounting, and reporting-date rules must be decided before implementing these calculations.

Authorized users can inspect the underlying records for each figure and select a custom date range. Report figures are permission-controlled. Comparisons and totals use consistent period boundaries and do not double-count original records and their corrections.

## 14 Accounts authentication and recovery

Each store has one designated owner with full control over store settings and staff permissions. Employees have individual store-associated accounts. V1 does not require a configurable hierarchy of named roles.

The owner grants and removes per-account permissions covering protected actions and sensitive information. Staff cannot change their own or other staff permissions. The owner may separately delegate creation, deactivation, and password reset of staff accounts. Delegation cannot reset or take over the owner through normal staff-management functions.

Users sign in with a username and password and can sign out. Temporary initial access or an administrative reset requires the employee to choose a new password before ordinary use. No user can view another user's existing password.

Users can change passwords after verifying their existing credentials. Forgotten staff passwords can be reset by the owner or a permitted account manager. The reset is recorded and existing sessions are invalidated.

Accounts support optional Time-based One-Time Password (TOTP) two-factor authentication through compatible authenticator applications. The owner can require it for staff. When enabled, normal login requires the password and an authenticator code. An ordinary authenticator code alone cannot reset the password.

Two-factor setup supplies single-use backup codes to store securely. A backup code substitutes for an unavailable authenticator code during login; it does not make the password optional. If staff lose both, the owner can authorize re-enrollment after identity verification.

The owner receives a dedicated single-use recovery key during setup, stored securely outside 4f4b. Successful owner recovery invalidates the key and supplies a replacement. The exact recovery procedure and failure handling must be reviewed before authentication implementation.

Signing out ends the current session. Account deactivation and account recovery invalidate affected sessions. Account creation, deactivation, administrative resets, and recovery actions are audited with user, affected account, and time. Passwords, authentication secrets, and recovery codes never appear in these logs. Authentication and recovery require protection against repeated guessing.

Email and phone verification are postponed. Customer contact data remains supported without making it a login or verified recovery channel.

## 15 Store settings and interface

- A clean responsive interface supports quick catalog navigation and checkout on desktop and touchscreen devices.
- V1 supports English and Uzbek. User-interface translation is distinct from user-entered product names.
- Each store selects a currency, including the planned USD and UZS options. V1 does not perform automatic currency conversion. A setting change must never relabel old financial records as another currency.
- Time zone, closing time, week-ending day, price and credit thresholds, refund policy, notification schedules, permissions, and optional feature use are store settings where specified above.
- Payment and stock quantities must be displayed with their unit or currency so the meaning of a number is unambiguous.

## 16 Decisions to resolve during implementation

The following decisions need explicit answers before the affected feature is implemented. They do not all have to be settled before starting the product catalog.

| Decision | Why it matters | Resolve before |
| --- | --- | --- |
| Initial owner provisioning for managed and self-hosted installations | Both delivery models need a secure way to create the first owner and identify the store at login | Account setup and authentication implementation |
| Cost valuation when restock prices change; treatment of faulty returns and discarded-stock losses | One product's editable cost alone cannot determine every historical stock cost or gross profit adjustment | Costed restocking and profitability reporting |
| Allocation of a payment across several outstanding sales | Per-sale balances and later refunds depend on which debt was settled | Later customer-payment recording |
| Allowed refund policy exceptions | Default rejection of damaged goods should be distinguished from the owner's permitted business exceptions | Refund implementation |
| Measured-source corrections, division losses, unusable remainders, and cancellations after physical division | Totals and portion composition must match physical stock after corrections and returns | Measured inventory operations |
| Price and cost basis for different ready-to-sell sizes; barcode-to-portion mapping | Identical products in different packages or sizes may need different commercial values or lookup targets | Portion pricing and scanning |
| Currency rounding, change, overpayments, and any split payment support | Checkout must distinguish sale value, amount tendered, payment received, and amount owed back | Payment finalization |
| Financial adjustment dates, overnight business-day boundaries, and saved report revisions | Corrected sale views and period activity reports must not double-count or silently shift history | Scheduled reporting |
| Notification channel and delivery behavior while the owner is offline | A saved in-app notification is different from an external push or email | Notification delivery |
| Owner recovery, staff reset boundaries, store identification at login, and session rules | Account recovery and delegation must not bypass the owner's protected position | Authentication implementation |
| Currency changes after financial activity exists | Existing records must retain their original meaning | Allowing a currency-setting change after use |
| Database and uploaded-file backup schedule, retention, and restoration procedure | The 4f4b provider operates backups for managed hosting; the installation operator is responsible for self-hosted backups | Real business use; restoration must be tested |
| Automated managed-service subscription signup and billing, if included | A subscription business model does not itself specify billing integration or service lifecycle behavior | Offering automated paid signup |

## 17 Pilot data and source distribution

The source repository contains reusable application code, schema migrations, documentation, safe example configuration, and fictional demonstration data. Secrets, the real store database, customer details, actual sales, and private uploads are excluded.

Self-hosting documentation must cover required services, initial configuration and owner setup, updates and database migrations, and backup and restoration responsibilities. A locally hosted installation is not a second database synchronized with the managed service; moving or synchronizing business data between deployments is not automatically part of the hosting choice.

Pilot product categories, brands, measured sizes, prices, and policies are configured as data. Another store can supply different data without a separate plumbing-specific codebase. The requirements describe the V1 release target; implementation proceeds in small usable increments.
