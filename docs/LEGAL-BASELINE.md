# EU and Italy legal baseline — applicability checklist (not legal advice)

CLARYEL must implement compliance-by-design and keep auditable evidence. **No claim of full GDPR/AI Act/CRA/NIS2 compliance is made.** Exact obligations depend on use case, sector, distribution, customer/controller/processor roles and legal assessment.

## GDPR (EU 2016/679)
- Articles 5, 6 and where applicable 9: purpose limitation, minimization, storage limits, lawful basis, sensitive-data safeguards and accountability. Technical approval for export is not automatically a GDPR legal basis.
- Articles 12–22: processes for requests, access, deletion, correction, portability and restrictions, with authenticated scope and retention controls.
- Article 25: data protection by design/default; zero private-to-cloud content by default.
- Articles 28/30: controller-processor and subprocessor assessment, suitable agreements, records of processing where required.
- Article 32: encryption, access control, confidentiality, resilience, recovery and recurring security testing.
- Articles 33–34: incident evaluation, escalation and notifications when required.
- Articles 35–36: risk-based DPIA decision; consultation when legally required.
- Chapter V: assess international data transfers, later cloud connections and external providers. Running two agents on one computer does not itself determine whether disclosure or processing by another party occurs.

## Other relevant instruments
- EU AI Act (Regulation 2024/1689, as amended): classify each deployment; assess prohibited/high-risk uses, provider/deployer responsibilities, transparency, records, oversight and applicable timelines.
- EU Cyber Resilience Act (Regulation 2024/2847): assess whether the product and distribution fall within scope; secure development, vulnerability response, update policy, documentation and applicable reporting.
- NIS2 and Italian implementing measures: check whether CLARYEL or a particular customer is in scope, including supply-chain obligations.
- Italy: GDPR plus applicable Codice Privacy provisions and Garante guidance; consult qualified counsel before commercial release.

## Evidence required before real client use
Data inventory, flows and recipients, tenant boundaries, lawful basis and role mapping, data retention/deletion, contract/DPA review, DPIA determination, incident playbook, recovery testing, external processor and cross-border-transfer review, vulnerability policy, versioned approvals and independent security review. Offline capability does not waive legal duties.
