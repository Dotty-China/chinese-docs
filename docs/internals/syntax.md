---
layout: doc-page
title: "Scala Syntax Summary"
---

The following descriptions of Scala tokens uses literal characters `‘c’` when
referring to the ASCII fragment `\u0000` – `\u007F`.

_Unicode escapes_ are used to represent the Unicode character with the given
hexadecimal code:

```ebnf
UnicodeEscape ::= ‘\’ ‘u’ {‘u’} hexDigit hexDigit hexDigit hexDigit
hexDigit      ::= ‘0’ | … | ‘9’ | ‘A’ | … | ‘F’ | ‘a’ | … | ‘f’
```

Informal descriptions are typeset as `“some comment”`.

### Lexical Syntax
The lexical syntax of Scala is given by the following grammar in EBNF
form.

```ebnf
whiteSpace       ::=  ‘\u0020’ | ‘\u0009’ | ‘\u000D’ | ‘\u000A’
upper            ::=  ‘A’ | … | ‘Z’ | ‘\$’ | ‘_’  “… and Unicode category Lu”
lower            ::=  ‘a’ | … | ‘z’ “… and Unicode category Ll”
letter           ::=  upper | lower “… and Unicode categories Lo, Lt, Nl”
digit            ::=  ‘0’ | … | ‘9’
paren            ::=  ‘(’ | ‘)’ | ‘[’ | ‘]’ | ‘{’ | ‘}’ | ‘'(’ | ‘'[’ | ‘'{’
delim            ::=  ‘`’ | ‘'’ | ‘"’ | ‘.’ | ‘;’ | ‘,’
opchar           ::=  “printableChar not matched by (whiteSpace | upper | lower |
                       letter | digit | paren | delim | opchar | Unicode_Sm |
                       Unicode_So)”
printableChar    ::=  “all characters in [\u0020, \u007F] inclusive”
charEscapeSeq    ::=  ‘\’ (‘b’ | ‘t’ | ‘n’ | ‘f’ | ‘r’ | ‘"’ | ‘'’ | ‘\’)

op               ::=  opchar {opchar}
varid            ::=  lower idrest
alphaid          ::=  upper idrest
                   |  varid
plainid          ::=  alphaid
                   |  op
id               ::=  plainid
                   |  ‘`’ { charNoBackQuoteOrNewline | UnicodeEscape | charEscapeSeq } ‘`’
                   |  INT                           // interpolation id, only for quasi-quotes
idrest           ::=  {letter | digit} [‘_’ op]

integerLiteral   ::=  (decimalNumeral | hexNumeral) [‘L’ | ‘l’]
decimalNumeral   ::=  ‘0’ | nonZeroDigit {digit}
hexNumeral       ::=  ‘0’ (‘x’ | ‘X’) hexDigit {hexDigit}
digit            ::=  ‘0’ | nonZeroDigit
nonZeroDigit     ::=  ‘1’ | … | ‘9’

floatingPointLiteral
                 ::=  digit {digit} ‘.’ {digit} [exponentPart] [floatType]
                   |  ‘.’ digit {digit} [exponentPart] [floatType]
                   |  digit {digit} exponentPart [floatType]
                   |  digit {digit} [exponentPart] floatType
exponentPart     ::=  (‘E’ | ‘e’) [‘+’ | ‘-’] digit {digit}
floatType        ::=  ‘F’ | ‘f’ | ‘D’ | ‘d’

booleanLiteral   ::=  ‘true’ | ‘false’

characterLiteral ::=  ‘'’ (printableChar | charEscapeSeq) ‘'’

stringLiteral    ::=  ‘"’ {stringElement} ‘"’
                   |  ‘"""’ multiLineChars ‘"""’
stringElement    ::=  printableChar \ (‘"’ | ‘\’)
                   |  UnicodeEscape
                   |  charEscapeSeq
multiLineChars   ::=  {[‘"’] [‘"’] char \ ‘"’} {‘"’}
processedStringLiteral
                 ::=  alphaid ‘"’ {printableChar \ (‘"’ | ‘$’) | escape} ‘"’
                   |  alphaid ‘"""’ {[‘"’] [‘"’] char \ (‘"’ | ‘$’) | escape} {‘"’} ‘"""’
escape           ::=  ‘$$’
                   |  ‘$’ letter { letter | digit }
                   |  ‘{’ Block  [‘;’ whiteSpace stringFormat whiteSpace] ‘}’
stringFormat     ::=  {printableChar \ (‘"’ | ‘}’ | ‘ ’ | ‘\t’ | ‘\n’)}

symbolLiteral    ::=  ‘'’ plainid

comment          ::=  ‘/*’ “any sequence of characters; nested comments are allowed” ‘*/’
                   |  ‘//’ “any sequence of characters up to end of line”

nl               ::=  “new line character”
semi             ::=  ‘;’ |  nl {nl}
```


## Context-free Syntax

The context-free syntax of Scala is given by the following EBNF
grammar:

### Literals and Paths
```ebnf
SimpleLiteral     ::=  [‘-’] integerLiteral
                    |  [‘-’] floatingPointLiteral
                    |  booleanLiteral
                    |  characterLiteral
                    |  stringLiteral
                    |  symbolLiteral
Literal           ::=  SimpleLiteral
                    |  processedStringLiteral
                    |  ‘null’

QualId            ::=  id {‘.’ id}
ids               ::=  id {‘,’ id}

Path              ::=  StableId
                    |  [id ‘.’] ‘this’
StableId          ::=  id
                    |  Path ‘.’ id
                    |  [id ‘.’] ‘super’ [ClassQualifier] ‘.’ id
ClassQualifier    ::=  ‘[’ id ‘]’
```

### Types
```ebnf
Type              ::=  [FunArgMods] FunArgTypes ‘=>’ Type                       Function(ts, t)
                    |  HkTypeParamClause ‘=>’ Type                              TypeLambda(ps, t)
                    |  InfixType
FunArgMods        ::=  { ‘implicit’ | ‘erased’ }
FunArgTypes       ::=  InfixType
                    |  ‘(’ [ FunArgType {‘,’ FunArgType } ] ‘)’
                    |  ‘(’ TypedFunParam {‘,’ TypedFunParam } ‘)’
TypedFunParam     ::=  id ‘:’ Type
InfixType         ::=  RefinedType {id [nl] RefinedType}                        InfixOp(t1, op, t2)
RefinedType       ::=  WithType {[nl] Refinement}                               RefinedTypeTree(t, ds)
WithType          ::=  AnnotType {‘with’ AnnotType}                             (deprecated)
AnnotType         ::=  SimpleType {Annotation}                                  Annotated(t, annot)
SimpleType        ::=  SimpleType TypeArgs                                      AppliedTypeTree(t, args)
                    |  SimpleType ‘#’ id                                        Select(t, name)
                    |  StableId
                    |  [‘-’ | ‘+’ | ‘~’ | ‘!’] StableId                         PrefixOp(expr, op)
                    |  Path ‘.’ ‘type’                                          SingletonTypeTree(p)
                    |  ‘(’ ArgTypes ‘)’                                         Tuple(ts)
                    |  ‘_’ TypeBounds
                    |  Refinement                                               RefinedTypeTree(EmptyTree, refinement)
                    |  SimpleLiteral                                            SingletonTypeTree(l)
ArgTypes          ::=  Type {‘,’ Type}
                    |  NamedTypeArg {‘,’ NamedTypeArg}
FunArgType        ::=  Type
                    |  ‘=>’ Type                                                PrefixOp(=>, t)
ParamType         ::=  [‘=>’] ParamValueType
ParamValueType    ::=  Type [‘*’]                                               PostfixOp(t, "*")
TypeArgs          ::=  ‘[’ ArgTypes ‘]’                                         ts
NamedTypeArg      ::=  id ‘=’ Type                                              NamedArg(id, t)
NamedTypeArgs     ::=  ‘[’ NamedTypeArg {‘,’ NamedTypeArg} ‘]’                  nts
Refinement        ::=  ‘{’ [RefineDcl] {semi [RefineDcl]} ‘}’                   ds
TypeBounds        ::=  [‘>:’ Type] [‘<:’ Type] | INT                            TypeBoundsTree(lo, hi)
TypeParamBounds   ::=  TypeBounds {‘<%’ Type} {‘:’ Type}                        ContextBounds(typeBounds, tps)
```

### Expressions
```ebnf
Expr              ::=  [FunArgMods] FunParams ‘=>’ Expr                         Function(args, expr), Function(ValDef([implicit], id, TypeTree(), EmptyTree), expr)
                    |  Expr1
BlockResult       ::=  [FunArgMods] FunParams ‘=>’ Block
                    |  Expr1
FunParams         ::=  Bindings
                    |  id
                    |  ‘_’
Expr1             ::=  ‘if’ ‘(’ Expr ‘)’ {nl} Expr [[semi] ‘else’ Expr]         If(Parens(cond), thenp, elsep?)
                    |  ‘if’ Expr ‘then’ Expr [[semi] ‘else’ Expr]               If(cond, thenp, elsep?)
                    |  ‘while’ ‘(’ Expr ‘)’ {nl} Expr                           WhileDo(Parens(cond), body)
                    |  ‘while’ Expr ‘do’ Expr                                   WhileDo(cond, body)
                    |  ‘do’ Expr [semi] ‘while’ Expr                            DoWhile(expr, cond)
                    |  ‘try’ Expr Catches [‘finally’ Expr]                      Try(expr, catches, expr?)
                    |  ‘try’ Expr [‘finally’ Expr]                              Try(expr, Nil, expr?)
                    |  ‘throw’ Expr                                             Throw(expr)
                    |  ‘return’ [Expr]                                          Return(expr?)
                    |  ForExpr
                    |  [SimpleExpr ‘.’] id ‘=’ Expr                             Assign(expr, expr)
                    |  SimpleExpr1 ArgumentExprs ‘=’ Expr                       Assign(expr, expr)
                    |  PostfixExpr [Ascription]
                    |  PostfixExpr ‘match’ ‘{’ CaseClauses ‘}’                  Match(expr, cases) -- point on match
Ascription        ::=  ‘:’ InfixType                                            Typed(expr, tp)
                    |  ‘:’ Annotation {Annotation}                              Typed(expr, Annotated(EmptyTree, annot)*)
Catches           ::=  ‘catch’ Expr
PostfixExpr       ::=  InfixExpr [id]                                           PostfixOp(expr, op)
InfixExpr         ::=  PrefixExpr
                    |  InfixExpr id [nl] InfixExpr                              InfixOp(expr, op, expr)
PrefixExpr        ::=  [‘-’ | ‘+’ | ‘~’ | ‘!’] SimpleExpr                       PrefixOp(expr, op)
SimpleExpr        ::=  ‘new’ Template                                           New(templ)
                    |  BlockExpr
                    |  ''{’ BlockExprContents ‘}’
                    |  ‘'(’ ExprsInParens ‘)’
                    |  ‘'[’ Type ‘]’
                    |  SimpleExpr1 [‘_’]                                        PostfixOp(expr, _)
SimpleExpr1       ::=  Literal
                    |  Path
                    |  ‘_’
                    |  ‘(’ ExprsInParens ‘)’                                    Parens(exprs)
                    |  SimpleExpr ‘.’ id                                        Select(expr, id)
                    |  SimpleExpr (TypeArgs | NamedTypeArgs)                    TypeApply(expr, args)
                    |  SimpleExpr1 ArgumentExprs                                Apply(expr, args)
                    |  XmlExpr
ExprsInParens     ::=  ExprInParens {‘,’ ExprInParens}
ExprInParens      ::=  PostfixExpr ‘:’ Type
                    |  Expr
ParArgumentExprs  ::=  ‘(’ ExprsInParens ‘)’                                    exprs
                    |  ‘(’ [ExprsInParens] PostfixExpr ‘:’ ‘_’ ‘*’ ‘)’          exprs :+ Typed(expr, Ident(wildcardStar))
ArgumentExprs     ::=  ParArgumentExprs
                    |  [nl] BlockExpr
BlockExpr         ::=  ‘{’ BlockExprContents ‘}’
BlockExprContents ::=  CaseClauses | Block
Block             ::=  {BlockStat semi} [BlockResult]                           Block(stats, expr?)
BlockStat         ::=  Import
                    |  {Annotation} [‘implicit’ | ‘lazy’] Def
                    |  {Annotation} {LocalModifier} TmplDef
                    |  Expr1

ForExpr           ::=  ‘for’ (‘(’ Enumerators ‘)’ | ‘{’ Enumerators ‘}’)        ForYield(enums, expr)
                       {nl} [‘yield’] Expr
                    |  ‘for’ Enumerators (‘do’ Expr | ‘yield’ Expr)             ForDo(enums, expr)
Enumerators       ::=  Generator {semi Enumerator | Guard}
Enumerator        ::=  Generator
                    |  Guard
                    |  Pattern1 ‘=’ Expr                                        GenAlias(pat, expr)
Generator         ::=  Pattern1 ‘<-’ Expr                                       GenFrom(pat, expr)
Guard             ::=  ‘if’ PostfixExpr

CaseClauses       ::=  CaseClause { CaseClause }                                Match(EmptyTree, cases)
CaseClause        ::=  ‘case’ (Pattern [Guard] ‘=>’ Block | INT)                CaseDef(pat, guard?, block)   // block starts at =>

Pattern           ::=  Pattern1 { ‘|’ Pattern1 }                                Alternative(pats)
Pattern1          ::=  PatVar ‘:’ RefinedType                                   Bind(name, Typed(Ident(wildcard), tpe))
                    |  Pattern2
Pattern2          ::=  [id ‘@’] InfixPattern                                    Bind(name, pat)
InfixPattern      ::=  SimplePattern { id [nl] SimplePattern }                  InfixOp(pat, op, pat)
SimplePattern     ::=  PatVar                                                   Ident(wildcard)
                    |  Literal                                                  Bind(name, Ident(wildcard))
                    |  ‘(’ [Patterns] ‘)’                                       Parens(pats) Tuple(pats)
                    |  XmlPattern
                    |  SimplePattern1 [TypeArgs] [ArgumentPatterns]
SimplePattern1    ::=  Path
                    |  SimplePattern1 ‘.’ id
PatVar            ::=  varid
                    |  ‘_’
Patterns          ::=  Pattern {‘,’ Pattern}
ArgumentPatterns  ::=  ‘(’ [Patterns] ‘)’                                       Apply(fn, pats)
                    |  ‘(’ [Patterns ‘,’] Pattern2 ‘:’ ‘_’ ‘*’ ‘)’
```

### Type and Value Parameters
```ebnf
ClsTypeParamClause::=  ‘[’ ClsTypeParam {‘,’ ClsTypeParam} ‘]’
ClsTypeParam      ::=  {Annotation} [‘+’ | ‘-’]                                   TypeDef(Modifiers, name, tparams, bounds)
                       id [HkTypeParamClause] TypeParamBounds                   Bound(below, above, context)

DefTypeParamClause::=  ‘[’ DefTypeParam {‘,’ DefTypeParam} ‘]’
DefTypeParam      ::=  {Annotation} id [HkTypeParamClause] TypeParamBounds

TypTypeParamClause::=  ‘[’ TypTypeParam {‘,’ TypTypeParam} ‘]’
TypTypeParam      ::=  {Annotation} id [HkTypeParamClause] TypeBounds

HkTypeParamClause ::=  ‘[’ HkTypeParam {‘,’ HkTypeParam} ‘]’
HkTypeParam       ::=  {Annotation} [‘+’ | ‘-’] (Id[HkTypeParamClause] | ‘_’)
                       TypeBounds

ClsParamClauses   ::=  {ClsParamClause} [[nl] ‘(’ [FunArgMods] ClsParams ‘)’]
ClsParamClause    ::=  [nl] ‘(’ [ClsParams] ‘)’
ClsParams         ::=  ClsParam {‘,’ ClsParam}
ClsParam          ::=  {Annotation}                                             ValDef(mods, id, tpe, expr) -- point of mods on val/var
                       [{Modifier} (‘val’ | ‘var’) | ‘inline’] Param
Param             ::=  id ‘:’ ParamType [‘=’ Expr]
                    |  INT

DefParamClauses   ::=  {DefParamClause} [[nl] ‘(’ [FunArgMods] DefParams ‘)’]
DefParamClause    ::=  [nl] ‘(’ [DefParams] ‘)’
DefParams         ::=  DefParam {‘,’ DefParam}
DefParam          ::=  {Annotation} [‘inline’] Param                            ValDef(mods, id, tpe, expr) -- point of mods at id.
```

### Bindings and Imports
```ebnf
Bindings          ::=  ‘(’ Binding {‘,’ Binding} ‘)’
Binding           ::=  (id | ‘_’) [‘:’ Type]                                    ValDef(_, id, tpe, EmptyTree)

Modifier          ::=  LocalModifier
                    |  AccessModifier
                    |  ‘override’
LocalModifier     ::=  ‘abstract’
                    |  ‘final’
                    |  ‘sealed’
                    |  ‘implicit’
                    |  ‘lazy’
AccessModifier    ::=  (‘private’ | ‘protected’) [AccessQualifier]
AccessQualifier   ::=  ‘[’ (id | ‘this’) ‘]’

Annotation        ::=  ‘@’ SimpleType {ParArgumentExprs}                        Apply(tpe, args)

Import            ::=  ‘import’ ImportExpr {‘,’ ImportExpr}
ImportExpr        ::=  StableId ‘.’ (id | ‘_’ | ImportSelectors)                Import(expr, sels)
ImportSelectors   ::=  ‘{’ {ImportSelector ‘,’} (ImportSelector | ‘_’) ‘}’
ImportSelector    ::=  id [‘=>’ id | ‘=>’ ‘_’]                                  Ident(name), Pair(id, id)
```

### Declarations and Definitions
```ebnf
RefineDcl         ::=  ‘val’ VarDcl
                    |  ‘def’ DefDcl
                    |  ‘type’ {nl} TypeDcl
                    |  INT
Dcl               ::=  RefineDcl
                    |  ‘var’ VarDcl
ValDcl            ::=  ids ‘:’ Type                                             PatDef(_, ids, tpe, EmptyTree)
VarDcl            ::=  ids ‘:’ Type                                             PatDef(_, ids, tpe, EmptyTree)
DefDcl            ::=  DefSig [‘:’ Type]                                        DefDef(_, name, tparams, vparamss, tpe, EmptyTree)
DefSig            ::=  id [DefTypeParamClause] DefParamClauses
TypeDcl           ::=  id [TypTypeParamClause] [‘=’ Type]                       TypeDefTree(_, name, tparams, tpt)
                    |  id [HkTypeParamClause] TypeBounds                        TypeDefTree(_, name, tparams, bounds)

Def               ::=  ‘val’ PatDef
                    |  ‘var’ VarDef
                    |  ‘def’ DefDef
                    |  ‘type’ {nl} TypeDcl
                    |  TmplDef
                    |  INT
PatDef            ::=  Pattern2 {‘,’ Pattern2} [‘:’ Type] ‘=’ Expr              PatDef(_, pats, tpe?, expr)
VarDef            ::=  PatDef
                    |  ids ‘:’ Type ‘=’ ‘_’
DefDef            ::=  DefSig [‘:’ Type] ‘=’ Expr                               DefDef(_, name, tparams, vparamss, tpe, expr)
                    |  DefSig [nl] ‘{’ Block ‘}’                                DefDef(_, name, tparams, vparamss, tpe, Block)
                    |  ‘this’ DefParamClause DefParamClauses                    DefDef(_, <init>, Nil, vparamss, EmptyTree, expr | Block)
                       (‘=’ ConstrExpr | [nl] ConstrBlock)

TmplDef           ::=  ([‘case’] ‘class’ | ‘trait’) ClassDef
                    |  [‘case’] ‘object’ ObjectDef
                    |  ‘enum’ EnumDef
ClassDef          ::=  id ClassConstr TemplateOpt                               ClassDef(mods, name, tparams, templ)
ClassConstr       ::=  [ClsTypeParamClause] [ConstrMods] ClsParamClauses         with DefDef(_, <init>, Nil, vparamss, EmptyTree, EmptyTree) as first stat
ConstrMods        ::=  {Annotation} [AccessModifier]
ObjectDef         ::=  id TemplateOpt                                           ModuleDef(mods, name, template)  // no constructor
EnumDef           ::=  id ClassConstr [‘extends’ [ConstrApps]] EnumBody         EnumDef(mods, name, tparams, template)
TemplateOpt       ::=  [‘extends’ Template | [nl] TemplateBody]
Template          ::=  ConstrApps [TemplateBody] | TemplateBody                 Template(constr, parents, self, stats)
ConstrApps        ::=  ConstrApp {‘with’ ConstrApp}
ConstrApp         ::=  AnnotType {ArgumentExprs}                                Apply(tp, args)
ConstrExpr        ::=  SelfInvocation
                    |  ConstrBlock
SelfInvocation    ::=  ‘this’ ArgumentExprs {ArgumentExprs}
ConstrBlock       ::=  ‘{’ SelfInvocation {semi BlockStat} ‘}’

TemplateBody      ::=  [nl] ‘{’ [SelfType] TemplateStat {semi TemplateStat} ‘}’ (self, stats)
TemplateStat      ::=  Import
                    |  {Annotation [nl]} {Modifier} Def
                    |  {Annotation [nl]} {Modifier} Dcl
                    |  Expr1
                    |
SelfType          ::=  id [‘:’ InfixType] ‘=>’                                  ValDef(_, name, tpt, _)
                    |  ‘this’ ‘:’ InfixType ‘=>’

EnumBody          ::=  [nl] ‘{’ [SelfType] EnumStat {semi EnumStat} ‘}’
EnumStat          ::=  TemplateStat
                    |  {Annotation [nl]} {Modifier} EnumCase
EnumCase          ::=  ‘case’ (id ClassConstr [‘extends’ ConstrApps]] | ids)

TopStatSeq        ::=  TopStat {semi TopStat}
TopStat           ::=  {Annotation [nl]} {Modifier} TmplDef
                    |  Import
                    |  Packaging
                    |  PackageObject
Packaging         ::=  ‘package’ QualId [nl] ‘{’ TopStatSeq ‘}’                 Package(qid, stats)
PackageObject     ::=  ‘package’ ‘object’ ObjectDef                             object with package in mods.

CompilationUnit   ::=  {‘package’ QualId semi} TopStatSeq                       Package(qid, stats)
```
