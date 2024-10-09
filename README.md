# beancount-auto-import
A package that provides the MixIn `AutoImporter` which can be used in any beancount importer to automatically change the Payee, the narrative and add a posting to a transaction.

This allows to easily process expenses that occur often and always need the same account. 

## Usage
In herit the `AutoImporter` class in addition to the `Importer` from `beangulp`
```python
class ECImporter(Importer, AutoImporter):
    def __init__(
        self,
        iban: str,
        account_name: str,
        user: str,
        file_encoding: Optional[str] = "ISO-8859-1",
    ):
        self.iban = _format_iban(iban)
        self.account_name = account_name
        self.user = user
        self.file_encoding = file_encoding
        self.import_rules = []

        self._date_from = None
        self._date_to = None
        self._line_index = -1
```
Then use the methods `load_import_rules` and `auto_fill_transaction` to change the transactions.
This should usually happen in the `extract` method of your importer.

The `load_import_rules` method requires the path to a file where your import rules are defined. This file could look like this:
```yaml
Rundfunkbeitrag: # name
  match:
    narration: [] # a list of regex-strings that should match on the original narration
    payee: # a list of regex expressions that should match on the original payee
    - Rundfunk ARD, ZDF, DRadio
  replacements:
    account: Expenses:TV # the account that should be added
    narration: Public Broadcasting # the new narration that should be inserted
    payee: '' # the new payee that should be inserted, Empty string if the original should be kept
Aldi:
  match:
    narration:
    - aldi
    payee:
    - aldi
  replacements:
    account: Expenses:Food
    narration: Groceries
    payee: Aldi
```
The `auto_fill_transaction` method will check each transaction for matches with any of the regex strings and if a match is found then it replaces the supplied values. 
