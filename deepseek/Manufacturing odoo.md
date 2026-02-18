

Introduction to Odoo Manufacturing for Engineers: A Technical Walkthrough

This guide provides an engineering perspective on setting up and executing a manufacturing process in Odoo. We will use a real product, the Bohopanna Grisha Romper, to illustrate every concept, from master data configuration to the completion of a manufacturing order.

Our goal is to transform this product listing data into a structured, repeatable, and traceable production process within Odoo.

1. Understanding the Product: The Grisha Romper (Data Modeling)

First, we analyze the product data from the website to define our master data in Odoo.

Product: GRISHA ROMPER
Type: Storable Product (as we will manufacture and stock it).
Composition: 100% COTTON
Compliance: SNI : NPB 1-136-003-230844-1 (This is an Indonesian National Standard certificate).
Variants: The product has two key attributes that create multiple variants:

· Color: CHOCO, DUSTY PINK, LIME, VANILLA
· Size: 0-6M, 6-12M, 1Y, 2Y, 3Y, 4Y, 5-6Y, 7-8Y, 9-10Y
  Physical Properties: Weight: 200 grams, Dimensions: 3 × 5 × 3 cm.

Engineering Setup in Odoo:

· Product Template: Create a product template named "GRISHA ROMPER".
· Attributes & Variants: Configure "Color" and "Size" as product attributes with their respective values. Odoo will automatically generate all 4 (colors) x 10 (sizes) = 40 unique product variants (e.g., "GRISHA ROMPER - CHOCO, 0-6M").
· Compliance Tracking: Add a custom field or use the "Quality Control" module to store the SNI certificate number at the product template level, ensuring it applies to all variants. This is crucial for regulated baby clothing.

2. Bill of Materials (BOM): The Product Recipe

The BOM is the core engineering document defining how the romper is made. For a garment like this, we use a Manufacturing BOM type.

BOM Structure for "GRISMA ROMPER - CHOCO, 0-6M" (Example Variant):

· Finished Product: GRISHA ROMPER (Variant: CHOCO, 0-6M)
· Type: Manufacture this product.
· BOM Lines (Components):
  1. Fabric: 100% Cotton Jersey. Quantity: 0.65 meters. (Calculated from the pattern nesting for a 0-6M size).
  2. Thread: Polyester Sewing Thread. Quantity: 50 meters. (Consumable good practice).
  3. Main Label: "Bohopanna" woven label. Quantity: 1 unit.
  4. Size/Care Label: Printed label with size (0-6M) and care instructions. Quantity: 1 unit.
  5. Hang Tag: Brand card with price and barcode. Quantity: 1 unit.
  6. Safety Pin: For attaching hang tag. Quantity: 1 unit.
  7. Polybag: Individual packaging. Quantity: 1 unit.
· Operations (Routings & Work Centers): Garment manufacturing follows a specific routing.
  1. Operation: Cutting
     · Work Center: Cutting Table (with die cutter or automated cutter).
     · Duration: 2 minutes per bundle (e.g., 20 layers).
  2. Operation: Sewing
     · Work Center: Industrial Sewing Machine Station.
     · Duration: 15 minutes per piece.
  3. Operation: Quality Control
     · Work Center: Inspection Table.
     · Checks: Verify seam strength, no loose threads, button/closure safety (critical for baby wear), correct label placement.
     · Duration: 3 minutes per piece.
  4. Operation: Finishing & Packing
     · Work Center: Packing Station.
     · Tasks: Attach hang tag, fold, pack into polybag.
     · Duration: 2 minutes per piece.

3. Creating a Manufacturing Order (MO)

Now, let's simulate the process of creating an MO to produce 100 units of the "GRISHA ROMPER - DUSTY PINK, 1Y".

1. Initiation: An MO can be created manually, from a Sales Order (MTO), or from a Minimum Stock Rule (Reorder Point).
2. MO Form: Go to Manufacturing → Operations → Manufacturing Orders → Create.
   · Product: Select "GRISHA ROMPER - DUSTY PINK, 1Y".
   · Quantity: 100 Units.
   · BOM: Odoo will automatically suggest the correct BOM for this specific variant.
   · Deadline: Set the planned start/end dates based on capacity.
3. Validation: Click "Confirm". This action:
   · Reserves Stock: Checks availability for all components (fabric, thread, labels, etc.) for this specific variant. If a component is missing, it may trigger a purchase order or internal transfer request.
   · Creates MO & Tracked Lots: The MO is now in "Confirmed" state, and a unique production lot number can be assigned for full traceability (vital for recalls).

4. Executing the Manufacturing Order

This follows the engineering routing defined earlier.

1. Start Production: Click "Start". The MO moves to "In Progress".
2. Process Operations: Workers use tablets or scan barcodes to:
   · Record Time & Quantity: At the cutting station, they record start/stop time and the number of pieces cut.
   · Consume Material: When cutting is finished, they confirm the exact fabric quantity used. This posts a inventory move, deducting 65 meters (0.65m * 100) from the fabric roll's stock location.
   · Quality Checks: At the QC station, if the defect rate exceeds a threshold (e.g., >2%), the operation can be blocked, and a quality alert (Non-Conformity) is generated.
3. Track WIP: Work-in-Progress is tracked at each operation, providing real-time visibility on the shop floor.

5. Completing the Order

1. Last Operation: The final operation "Finishing & Packing" is marked as done.
2. Produce: Click the "Produce" or "Mark as Done" button.
   · Final Product Receipt: A inventory move is created to receive 100 units of the finished "GRISHA ROMPER - DUSTY PINK, 1Y" into the "Finished Goods" location.
   · Cost Calculation: Odoo calculates the actual production cost based on component consumption and operation times (if using work center costing).
   · Traceability: The entire production batch is now linked to the lot number, including which fabric roll (with its own lot/supplier) was used and which operator performed each operation.

Key Takeaways for Engineers

· Data Integrity: The entire system's reliability depends on the accuracy of the master data (BOMs, Routings) and inventory transactions.
· Automation: Odoo automates stock valuation, material reservation, and cost tracking, reducing manual errors.
· Traceability: For a regulated product like baby clothing, the system provides end-to-end traceability from raw material supplier to finished good customer, which is essential for compliance (like the SNI certificate shown).
· Scalability: This setup allows Bohopanna to efficiently manage the production complexity of 40 SKUs, plan capacity, and scale their operations.

This framework provides a robust foundation. Further extensions, such as the "Baby Clothing Manufacturing Management" addon idea from our previous discussion, could add features like automated nesting or enhanced compliance reporting.