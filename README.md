# Carbon Calculation Model


    databaseChangeLog:
    - changeSet:
        id: cbam-mvp-schema
        author: Enkodia Spring
        changes:
          - createTable:
              tableName: company
              remarks: |
                Legal entity participating in CBAM reporting.
                Examples:
                - legal_name: "ACME Steel Sp. z o.o."
                - registration_number: "KRS 0000123456"
                - country_code: "PL"
                - type: IMPORTER | EXPORTER | SUPPLIER | AUDITOR
              columns:
                - column:
                    name: id
                    type: UUID
                    constraints:
                      primaryKey: true
                - column:
                    name: legal_name
                    type: VARCHAR(255)
                    remarks: "Official registered legal name of the company."
                - column:
                    name: registration_number
                    type: VARCHAR(100)
                    remarks: "National business registration number."
                - column:
                    name: country_code
                    type: VARCHAR(10)
                    remarks: "ISO country code."
                - column:
                    name: type
                    type: VARCHAR(50)
                    remarks: "Role of the company in CBAM process."

        - createTable:
            tableName: application_user
            remarks: |
              User account belonging to a company.
              Examples:
              - email: "john.smith@acme.com"
              - first_name: "John"
              - last_name: "Smith"
              - status: ACTIVE | INVITED | BLOCKED
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: company_id
                  type: UUID
                  constraints:
                    nullable: false
                    foreignKeyName: fk_user_company
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: email
                  type: VARCHAR(255)
                  remarks: "User email used for authentication."
              - column:
                  name: first_name
                  type: VARCHAR(100)
                  remarks: "User first name."
              - column:
                  name: last_name
                  type: VARCHAR(100)
                  remarks: "User last name."
              - column:
                  name: status
                  type: VARCHAR(50)
                  remarks: "Account lifecycle status."

        - createTable:
            tableName: license
            remarks: |
              Commercial license assigned to a company.
              Examples:
              - license_key: "LIC-2026-ACME-0001"
              - valid_until: 2026-12-31
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_license_company
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: license_key
                  type: VARCHAR(255)
                  remarks: "Unique commercial license key."
              - column:
                  name: valid_until
                  type: DATE
                  remarks: "License expiration date."

        - createTable:
            tableName: site
            remarks: |
              Physical industrial site operated by a company.
              Examples:
              - name: "Plant Katowice"
              - address: "ul. Hutnicza 1, Katowice"
              - latitude: 50.2649
              - longitude: 19.0238
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_site_company
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: name
                  type: VARCHAR(255)
                  remarks: "Human-readable site name."
              - column:
                  name: address
                  type: TEXT
                  remarks: "Physical address of the site."
              - column:
                  name: latitude
                  type: DECIMAL(9,6)
                  remarks: "Geographic latitude."
              - column:
                  name: longitude
                  type: DECIMAL(9,6)
                  remarks: "Geographic longitude."

        - createTable:
            tableName: installation_type
            remarks: |
              Regulatory classification of industrial installations.
              Examples:
              - BF (blast furnace)
              - EAF (electric arc furnace)
              - CEM_KILN (cement kiln)
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: code
                  type: VARCHAR(100)
                  remarks: "Regulatory or technical installation type code."
              - column:
                  name: name
                  type: VARCHAR(255)
                  remarks: "Human-readable installation type name."

        - createTable:
            tableName: installation
            remarks: |
              Industrial installations belonging to a company and located at a specific site.
              Represents the emission source boundary for CBAM calculations.
              Examples:
              - Installation: BF-01 at Plant Katowice
              - Installation: EAF-02 at Plant Krakow
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: site_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_installation_site
                    referencedTableName: site
                    referencedColumnNames: id
              - column:
                  name: installation_type_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_installation_type
                    referencedTableName: installation_type
                    referencedColumnNames: id
              - column:
                  name: has_cems
                  type: BOOLEAN
                  remarks: "Indicates presence of continuous emission monitoring system."

        - createTable:
            tableName: cn_code
            remarks: |
              Combined Nomenclature codes used for CBAM classification.
              Examples:
              - 7207 (semi-finished steel products)
              - 2523 (cement clinker)
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: code
                  type: VARCHAR(50)
                  remarks: "Official CN code."
              - column:
                  name: description
                  type: TEXT
                  remarks: "CN code description."

        - createTable:
            tableName: unit_of_measurement
            remarks: |
              Standard measurement units.
              Examples:
              - KG
              - T
              - MWh
              - GJ
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: code
                  type: VARCHAR(50)
                  remarks: "Unit code."
              - column:
                  name: description
                  type: TEXT
                  remarks: "Unit description."

        - createTable:
            tableName: emission_factor
            remarks: |
              Emission factors used in CBAM calculations.
              Can represent electricity grid factors, fuel combustion factors,
              process-based factors or EU default values.
              Examples:
              - Electricity PL grid: 0.721 tCO2e/MWh
              - Natural gas combustion: 56.1 kgCO2e/GJ
              - EU default steel slab: 2.1 tCO2e/t
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: name
                  type: VARCHAR(255)
                  remarks: "Human-readable emission factor name."
              - column:
                  name: factor_kind
                  type: VARCHAR(100)
                  remarks: "Classification of emission factor."
              - column:
                  name: unit
                  type: VARCHAR(50)
                  remarks: "Unit of the emission factor."
              - column:
                  name: value
                  type: NUMERIC
                  remarks: "Numeric coefficient used in emission calculations."
              - column:
                  name: country_code
                  type: VARCHAR(10)
                  remarks: "ISO country code."
              - column:
                  name: source
                  type: VARCHAR(255)
                  remarks: "Origin of the emission factor."

        - createTable:
            tableName: produced_item
            remarks: |
              Product definition (nomenclature) produced at a site.
              Examples:
              - Steel slab S235
              - Cement clinker CEM I
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: site_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_item_site
                    referencedTableName: site
                    referencedColumnNames: id
              - column:
                  name: cn_code_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_item_cn
                    referencedTableName: cn_code
                    referencedColumnNames: id
              - column:
                  name: name
                  type: VARCHAR(255)
                  remarks: "Internal or commercial product name."

        - createTable:
            tableName: production_output
            remarks: |
              Production throughput per installation and product.
              One product can be produced on multiple installations.
              Examples:
              - 60 t of Steel slab from BF-01
              - 40 t of Steel slab from BF-02
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: installation_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_output_installation
                    referencedTableName: installation
                    referencedColumnNames: id
              - column:
                  name: produced_item_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_output_item
                    referencedTableName: produced_item
                    referencedColumnNames: id
              - column:
                  name: unit_of_measurement_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_output_uom
                    referencedTableName: unit_of_measurement
                    referencedColumnNames: id
              - column:
                  name: quantity
                  type: NUMERIC
                  remarks: "Produced quantity."
              - column:
                  name: activity_date
                  type: DATE
                  remarks: "Date of production."

        - createTable:
            tableName: transaction
            remarks: |
              Import/export transaction relevant for CBAM.
              Examples:
              - customs_declaration_code: "PL123456789"
              - period: "2026-Q1"
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: importer_company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_tx_importer
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: exporter_company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_tx_exporter
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: customs_declaration_code
                  type: VARCHAR(100)
                  remarks: "Customs declaration identifier."
              - column:
                  name: description
                  type: TEXT
                  remarks: "Transaction description."
              - column:
                  name: period
                  type: VARCHAR(50)
                  remarks: "Reporting period."

        - createTable:
            tableName: transaction_item
            remarks: |
              Individual goods within a transaction.
              Linked to concrete production output for traceability.
              Examples:
              - 60 t from BF-01
              - 40 t from BF-02
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: transaction_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_tx_item_tx
                    referencedTableName: transaction
                    referencedColumnNames: id
              - column:
                  name: production_output_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_tx_item_output
                    referencedTableName: production_output
                    referencedColumnNames: id
              - column:
                  name: quantity
                  type: NUMERIC
                  remarks: "Imported quantity."
              - column:
                  name: unit_price
                  type: NUMERIC
                  remarks: "Unit price."
              - column:
                  name: currency
                  type: VARCHAR(3)
                  remarks: "ISO currency code."

        - createTable:
            tableName: carbon_declaration
            remarks: |
              Carbon declaration snapshot submitted for CBAM.
              Examples:
              - period: 2026-Q1
              - status: DRAFT | SUBMITTED | ACCEPTED
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: importer_company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_decl_importer
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: exporter_company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_decl_exporter
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: period
                  type: VARCHAR(50)
                  remarks: "Reporting period."
              - column:
                  name: status
                  type: VARCHAR(50)
                  remarks: "Declaration status."
              - column:
                  name: version
                  type: INTEGER
                  remarks: "Declaration version."
              - column:
                  name: submitted_at
                  type: TIMESTAMP
                  remarks: "Submission timestamp."

        - createTable:
            tableName: declaration_history
            remarks: |
              Status transition history of declarations.
              Examples:
              - DRAFT → SUBMITTED → ACCEPTED
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: declaration_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_decl_history
                    referencedTableName: carbon_declaration
                    referencedColumnNames: id
              - column:
                  name: status
                  type: VARCHAR(50)
                  remarks: "Declaration status."
              - column:
                  name: description
                  type: TEXT
                  remarks: "Explanation or comment."
              - column:
                  name: changed_at
                  type: TIMESTAMP
                  remarks: "Timestamp of change."

        - createTable:
            tableName: declaration_audit
            remarks: |
              Audit outcome for a declaration.
              Examples:
              - result: APPROVED | REJECTED
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: declaration_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_decl_audit
                    referencedTableName: carbon_declaration
                    referencedColumnNames: id
              - column:
                  name: auditor_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_audit_user
                    referencedTableName: application_user
                    referencedColumnNames: id
              - column:
                  name: result
                  type: VARCHAR(50)
                  remarks: "Audit result."
              - column:
                  name: comment
                  type: TEXT
                  remarks: "Auditor comment."

        - createTable:
            tableName: evidence_request
            remarks: |
              Request for supporting evidence exchanged between companies.
              Examples:
              - Request for energy consumption data
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: declaration_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_evidence_decl
                    referencedTableName: carbon_declaration
                    referencedColumnNames: id
              - column:
                  name: requester_company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_evidence_requester
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: supplier_company_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_evidence_supplier
                    referencedTableName: company
                    referencedColumnNames: id
              - column:
                  name: status
                  type: VARCHAR(50)
                  remarks: "Request status."
              - column:
                  name: due_date
                  type: DATE
                  remarks: "Response deadline."
              - column:
                  name: description
                  type: TEXT
                  remarks: "Evidence request description."

        - createTable:
            tableName: formula
            remarks: |
              Calculation formula definition used by the calculation engine.
              Examples:
              - emissions = quantity * emission_factor.value
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: installation_type_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_formula_installation_type
                    referencedTableName: installation_type
                    referencedColumnNames: id
              - column:
                  name: scope
                  type: VARCHAR(50)
                  remarks: "GHG scope."
              - column:
                  name: expression
                  type: TEXT
                  remarks: "Calculation expression."

        - createTable:
            tableName: allocation_method
            remarks: |
              Allocation methodology for distributing emissions to products.
              Examples:
              - MASS_BASED
              - ENERGY_BASED
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: formula_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_allocation_formula
                    referencedTableName: formula
                    referencedColumnNames: id
              - column:
                  name: type
                  type: VARCHAR(50)
                  remarks: "Allocation method identifier."

        - createTable:
            tableName: scope_calculation
            remarks: |
              Calculated emissions per installation and scope.
              Examples:
              - SCOPE_1
              - SCOPE_2
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: declaration_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_scope_decl
                    referencedTableName: carbon_declaration
                    referencedColumnNames: id
              - column:
                  name: installation_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_scope_installation
                    referencedTableName: installation
                    referencedColumnNames: id
              - column:
                  name: scope
                  type: VARCHAR(50)
                  remarks: "GHG scope."
              - column:
                  name: emissions_total
                  type: NUMERIC
                  remarks: "Calculated emissions."

        - createTable:
            tableName: allocation_calculation
            remarks: |
              Allocated emissions per production output.
              Examples:
              - allocation_method: MASS_BASED
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: scope_calculation_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_alloc_scope
                    referencedTableName: scope_calculation
                    referencedColumnNames: id
              - column:
                  name: production_output_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_alloc_output
                    referencedTableName: production_output
                    referencedColumnNames: id
              - column:
                  name: allocation_method_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_alloc_method
                    referencedTableName: allocation_method
                    referencedColumnNames: id
              - column:
                  name: allocated_emissions
                  type: NUMERIC
                  remarks: "Allocated emissions value."

        - createTable:
            tableName: uploaded_file
            remarks: |
              Files uploaded to the system (inputs, evidence, exports).
              Examples:
              - production.xlsx
              - electricity.csv
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: file_name
                  type: VARCHAR(255)
                  remarks: "Original file name."
              - column:
                  name: file_type
                  type: VARCHAR(100)
                  remarks: "File MIME type."
              - column:
                  name: uploaded_at
                  type: TIMESTAMP
                  remarks: "Upload timestamp."

        - createTable:
            tableName: chat_message
            remarks: |
              Contextual communication messages.
              Examples:
              - Request clarification from exporter
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: context_type
                  type: VARCHAR(50)
                  remarks: "Context entity type."
              - column:
                  name: context_id
                  type: UUID
                  remarks: "Context entity identifier."
              - column:
                  name: sender_user_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_message_user
                    referencedTableName: application_user
                    referencedColumnNames: id
              - column:
                  name: content
                  type: TEXT
                  remarks: "Message content."
              - column:
                  name: created_at
                  type: TIMESTAMP
                  remarks: "Message creation timestamp."

        - createTable:
            tableName: audit_log
            remarks: |
              System-wide audit trail for compliance and security.
              Examples:
              - USER_LOGIN
              - DECLARATION_SUBMITTED
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: event_type
                  type: VARCHAR(100)
                  remarks: "Type of audited event."
              - column:
                  name: entity_type
                  type: VARCHAR(100)
                  remarks: "Affected entity type."
              - column:
                  name: entity_id
                  type: UUID
                  remarks: "Affected entity identifier."
              - column:
                  name: created_at
                  type: TIMESTAMP
                  remarks: "Event timestamp."
              - column:
                  name: created_by
                  type: VARCHAR(255)
                  remarks: "Actor initiating the event."

        - createTable:
            tableName: material
            remarks: |
              Material inputs consumed by an installation.
              Used as activity data for Scope 1 / Scope 3.1 calculations.
              Examples:
              - Iron ore
              - Limestone
              - Scrap steel
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: installation_id
                  type: UUID
                  constraints:
                    nullable: false
                    foreignKeyName: fk_material_installation
                    referencedTableName: installation
                    referencedColumnNames: id
              - column:
                  name: emission_factor_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_material_emission_factor
                    referencedTableName: emission_factor
                    referencedColumnNames: id
              - column:
                  name: name
                  type: VARCHAR(255)
                  remarks: "Material name or classification."
              - column:
                  name: quantity
                  type: NUMERIC
                  remarks: "Consumed material quantity."
              - column:
                  name: unit_of_measurement_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_material_uom
                    referencedTableName: unit_of_measurement
                    referencedColumnNames: id
              - column:
                  name: activity_date
                  type: DATE
                  remarks: "Date of material consumption."

        - createTable:
            tableName: energy
            remarks: |
              Energy consumption data per installation.
              Used for Scope 1 (fuels) and Scope 2 (electricity) calculations.
              Examples:
              - Electricity consumption
              - Natural gas combustion
            columns:
              - column:
                  name: id
                  type: UUID
                  constraints:
                    primaryKey: true
              - column:
                  name: installation_id
                  type: UUID
                  constraints:
                    nullable: false
                    foreignKeyName: fk_energy_installation
                    referencedTableName: installation
                    referencedColumnNames: id
              - column:
                  name: emission_factor_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_energy_emission_factor
                    referencedTableName: emission_factor
                    referencedColumnNames: id
              - column:
                  name: energy_type
                  type: VARCHAR(100)
                  remarks: "Energy classification (e.g. ELECTRICITY, NATURAL_GAS)."
              - column:
                  name: quantity
                  type: NUMERIC
                  remarks: "Consumed energy amount."
              - column:
                  name: unit_of_measurement_id
                  type: UUID
                  constraints:
                    foreignKeyName: fk_energy_uom
                    referencedTableName: unit_of_measurement
                    referencedColumnNames: id
              - column:
                  name: activity_date
                  type: DATE
                  remarks: "Date of energy consumption."

        - createTable:
            tableName: jt_declaration_item
            remarks: |
              Join table linking transaction items to a carbon declaration snapshot.
              Enables versioned, auditable declaration composition.
            columns:
              - column:
                  name: transaction_item_id
                  type: UUID
                  constraints:
                    nullable: false
                    foreignKeyName: fk_jt_decl_item_tx_item
                    referencedTableName: transaction_item
                    referencedColumnNames: id
              - column:
                  name: declaration_id
                  type: UUID
                  constraints:
                    nullable: false
                    foreignKeyName: fk_jt_decl_item_declaration
                    referencedTableName: carbon_declaration
                    referencedColumnNames: id
