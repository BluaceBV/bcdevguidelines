# Bluace development guidelines for Business Central

## AL Development

### Using the development branch

### CodeCops

CodeCops are enabled, warnings that can be avoided must be solved.

### Object centric programming

Table(Extension) with procedure, helper codeunit

### Object numbering

Where to use suffixes an when not?

### Naming

#### Objects

#### Files

#### Fields

#### Controls

#### Keys
#### Labels

### Use method codeunits for actions

Make sure that you use a method codeunits that can be called from the object, decide whether confirmation must be shown. No globals if not necessary.

### Creating event publishers

Always make sure that errors will be generated based on permission of the user.

### Creating event subscribers

Use as less parameters as needed. Not necessary to use the complete procedure description.

### Using event subscribers

IsTemporary must be used. 
The next step must be a call to a procedure on the object/entity
Subscriber codeunits must be 

### RequestPages

Use <Name>Control as control name

### Promoted actions

Use _Promoted

### Using ApplicationArea

### Update permission sets

For all products a permission set is included with all possible permissions on the objects that are part of the extension. Make sure that it's updated after each modification. Use AL-command, watch out for Pragma.

### Captions/Tooltips

### Translations using XLIFF Sync

Always supply nl-NL and en-US translations

### Remove unused parts

Make sure that no commented code is in the branch when pull requesting
Make sure that no unused elements are part of the code that must be integrated into the solutions.

### Compiler directives

Pragma

### Confirm Management

Use confirm management codeunit instead of System.Confirm directly.

### Dialogs

How to handle performance in combination with dialogs

[:arrow_left:](../README.md) [Back](../README.md)

