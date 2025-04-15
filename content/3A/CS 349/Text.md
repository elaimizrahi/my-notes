Text is a series of characters, and we need **standardized** encodings for characters in binary

**ASCII** - American standard code for information interchange
- standardized encoding page for symbols

**Unicode** - a superset of ASCII
- requires more than 1 byte to encode
- written as **U+** then a 4-digit hex (ie **H** is **U+0048** instead of decimal 72 in ASCII)
- The implementation of unicode is not defined by unicode

**UTF-8** - Universal Character Set Transformation Format 8 bit
- uses a multi-byte variable width encoding 

**Internationalization (i18n)** is designing and developing software so it can be adapted to different cultures and languages

**Localization (l10n)** is the act of implementing i18n. Locale means the region and language, e.g. en_CA or fr_CA.

**Implementing Localization**: 
1. Use HTML to identify i18n elements `<label data-i18n="label-name" ...>`
2. Create JSON translation data structures `{ en: {label-name: "Name" ... fr: {label-name: "Nom" ... `
3. Use preferred browser locale `navigator.language`
4. add locale switcher (dropdown to pick country/locale)

#### **Form Validation** 
interfaces have to validate text input typed by the user
- a required first
- certain format
- within a certain range
- unique
This is since the system needs the right data in the right format or the logic won't work. It also guides users (use secure passwords) and protects the system

**SQL Injection Attacks** - when user puts SQL into input form and tries to expose or harm the database

ie if username input then prompts db with `SELECT * FROM users WHERE name = '${userName}';`
but the user inputs `a'; DROP TABLE users; SELECT * FROM userinfo WHERE 't' = 't`
then this will utterly destroy the db and reveal user info, as it was still a valid SQL statement

- place error messages near fields, use colours
- allow different formats if possible
- same with filtering invalid chars
- and same with validating fields, do them when possible
	- This can be done with regex for text fields, for example

**Input formatting:** when form text is formatted as it's typed (ie for phone numbers)
**Input masking** provides a graphical representation of the final format and fills it in as the user types (phone numbers, date MM/DD/YYYY)



