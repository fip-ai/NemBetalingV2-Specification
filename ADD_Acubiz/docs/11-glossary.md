# Glossary

| Term | Meaning | Notes for rebuild |
|------|---------|-------------------|
| Acubiz CSV Import | inbound expense transaction file import scenario | creates general journal lines |
| Acubiz Pictures Import | inbound receipt-image import scenario | creates incoming document attachments |
| Acubiz Employee Export | outbound employee workbook generation scenario | uploads Excel workbook to Azure |
| Import / Email to Continia | blob-forwarding scenario using email delivery to Continia | there are three separately configurable channels |
| Azure config | per-scenario storage and routing settings | one row per integration type |
| Isolated password | Azure shared key stored in isolated storage | must not become a plain data field |
| GLN check | XML endpoint-ID validation against company GLN | used to prevent misrouted invoice forwarding |
| Document category | Continia configuration that provides recipient email address | drives email forwarding |
| Incoming document | BC record representing an imported supporting document | linked to journal line when receipt image exists |
| Approver (Godkender) | shared Acubiz approver/master-data record | used in employee export enrichment |
| Department mapping | shared translation from source department text to Acubiz export value | external dependency from shared app |
| Singleton setup | one-record application setup pattern | current app uses implicit setup creation |
| Clean-room rebuild | rebuild from specification without reading original source during implementation | desired future workflow |
