# Glossary

| Term | Meaning | Notes for rebuild |
|------|---------|-------------------|
| Collectia | External debt-collection service provider (formerly Kredinor) | use "Collectia" consistently in rebuilt code |
| Kredinor | Legacy name for Collectia used in original code identifiers | rename all identifiers during rebuild |
| INK (Inkasso) | Debt-collection case mode | full collection process at Collectia |
| RYK (Rykkerservice) | Reminder-service case mode | lighter reminder process at Collectia |
| Summons text (stævningstekst) | Description of the claim sent with the case | mandatory for every case |
| Course (forløb) | Processing workflow/route at Collectia | code sent with case creation |
| Session token (billet) | Authentication token from SOAP login | used for all subsequent requests in a workflow |
| FAK | Invoice amount type at Collectia | used for invoice documents |
| BET | Payment amount type at Collectia | used for payment documents |
| KRE | Credit amount type at Collectia | used for credit memos and refunds |
| RNT | Interest/finance charge amount type at Collectia | used for finance charge memos |
| GEB | Fee amount type at Collectia | used for reminder fees and additional fees |
| PER | Person debtor type | used when customer has no VAT number |
| VRK | Company debtor type (virksomhed) | used when domestic customer has a VAT number |
| UDL | Foreign debtor type (udland) | used for non-Danish customers |
| Issued reminder | BC document created when a reminder is issued | the primary record that triggers collection |
| Reminder level | Configuration per reminder-terms step | controls when automatic collection is triggered |
| Customer blocking | Setting Blocked = All on a BC customer | prevents all transactions for the customer |
| Activity log | BC standard logging mechanism | used for SOAP request troubleshooting |
