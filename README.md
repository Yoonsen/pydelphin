# pyDelphin <br/> Python libraries for DELPH-IN

> NOTE for previous pyDelphin users: The latest version (0.3) has some
> backward-**incompatible** changes that may break your code. The
> upgrade path shouldn't be difficult (contact me if there's trouble),
> and the changes bring big performance gains, so I think it's worth
> it. Sorry for the trouble while the API stabilizes.
> 
> Also note that the primary repo for pyDelphin is now
> https://github.com/delph-in/pydelphin

pyDelphin is a set of Python libraries for the
processing of [DELPH-IN](http://delph-in.net) data. It doesn't aim to
do heavy tasks like parsing or treebanking, but rather to provide Python
modules for loading a variety of DELPH-IN formats, such as [[incr
tsdb()]](http://www.delph-in.net/itsdb/) profiles or [Minimal Recursion
Semantics](http://moin.delph-in.net/RmrsTop) representations. These
modules offer a programmatic interface to the data to enable developers
or researchers to boostrap their own tools without having to re-invent
the wheel. pyDelphin also provides a [front-end tool][] for
accomplishing some tasks such as refreshing [incr tsdb()] profiles to a new
schema, creating sub-profiles, or converting between MRS representations
(SimpleMRS, MRS XML, DMRS, etc.).

* [Documentation](#documentation)
* [Usage Examples](#usage-examples)
* [Installation and Requirements](#installation-and-requirements)
* [Libraries](#sub-packages)

[front-end tool]: https://github.com/goodmami/pydelphin/wiki/Command-line-Tutorial

#### Documentation

API documentation is developing on the
[wiki](https://github.com/goodmami/pydelphin/wiki). Help is
appreciated!

#### Usage Examples

Here's a brief example of using the `itsdb` library:

```python
>>> from delphin import itsdb
>>> prof = itsdb.ItsdbProfile('/home/goodmami/logon/dfki/jacy/tsdb/gold/mrs')
>>> for row in prof.read_table('item'):
...     print(row.get('i-input'))
雨 が 降っ た ．
太郎 が 吠え た ．
窓 が 開い た ．
太郎 が 次郎 を 追っ た ．
[...]
>>> next(prof.read_table('result')).get('derivation')
'(utterance-root (91 utterance_rule-decl-finite -0.723739 0 4 (90 head_subj_rule -1.05796 0 4 (87 hf-complement-rule -0.50201 0 2 (86 quantify-n-rule -0.32216 0 1 (5 ame-noun 0 0 1 ("雨" 1 "\\"雨\\""))) (6 ga 0.531537 1 2 ("が" 2 "\\"が\\""))) (89 vstem-vend-rule -0.471785 2 4 (88 t-lexeme-c-stem-infl-rule 0.120963 2 3 (14 furu_1 0 2 3 ("降っ" 3 "\\"降っ\\""))) (24 ta-end -0.380719 3 4 ("た" 4 "\\"た\\""))))))'
```

Here's an example of loading a SimpleMRS representation:

```python
>>> from delphin.codecs import simplemrs
>>> m = simplemrs.loads_one('[ LTOP: h1 INDEX: e2 [ e TENSE: PAST MOOD: INDICATIVE PROG: - PERF: - SF: PROP ASPECT: DEFAULT_ASPECT PASS: - ] RELS: < [ udef_q_rel<0:1> LBL: h3 ARG0: x6 [ x PERS: 3 ] RSTR: h5 BODY: h4 ] [ "_ame_n_rel"<0:1> LBL: h7 ARG0: x6 ] [ "_furu_v_1_rel"<2:3> LBL: h8 ARG0: e2 ARG1: x6 ] > HCONS: < h5 qeq h7 > ]')
>>> m.ltop
'h1'
>>> for p in m.preds():
...     print('{}|{}|{}|{}'.format(p.string, p.lemma, p.pos, p.sense))
... 
udef_q_rel|udef|q|None
"_ame_n_rel"|ame|n|None
"_furu_v_1_rel"|furu|v|1
>>> print(simplemrs.dumps_one(m, pretty_print=True))
[ TOP: h1
  INDEX: e2 [ e TENSE: PAST MOOD: INDICATIVE PROG: - PERF: - SF: PROP ASPECT: DEFAULT_ASPECT PASS: - ]
  RELS: < [ udef_q_rel<0:1> LBL: h3 ARG0: x6 [ x PERS: 3 ] RSTR: h5 BODY: h4 ]
          [ "_ame_n_rel"<0:1> LBL: h7 ARG0: x6 ]
          [ "_furu_v_1_rel"<2:3> LBL: h8 ARG0: e2 ARG1: x6 ] >
  HCONS: < h5 qeq h7 > ]

```

Here is TDL introspection:

```python
>>> from delphin import tdl
>>> f = open('/home/goodmami/logon/lingo/erg/fundamentals.tdl', 'r')
>>> types = {t.identifier: t for t in tdl.parse(f)}
>>> types['basic_word'].supertypes
['word_or_infl_rule', 'word_or_punct_rule']
>>> types['basic_word'].features()
[('SYNSEM', <TdlDefinition object at 140559634120136>), ('TOKENS', <TdlDefinition object at 140559631479864>), ('ORTH', <TdlDefinition object at 140559631479000>)]
>>> types['basic_word'].coreferences
[('#rb', ['ORTH.RB', 'TOKENS.+LAST.+TRAIT.+RB']), ('#to', ['SYNSEM.LKEYS.KEYREL.CTO', 'ORTH.TO', 'TOKENS.+LAST.+TO']), ('#lb', ['ORTH.LB', 'TOKENS.+LIST.FIRST.+TRAIT.+LB']), ('#form', ['ORTH.FORM', 'TOKENS.+LIST.FIRST.+FORM']), ('#tl', ['SYNSEM.PHON.ONSET.--TL', 'TOKENS.+LIST']), ('#from', ['SYNSEM.LKEYS.KEYREL.CFROM', 'ORTH.FROM', 'TOKENS.+LIST.FIRST.+FROM']), ('#class', ['ORTH.CLASS', 'TOKENS.+LIST.FIRST.+CLASS'])]

```

And here's how to compile, parse, and generate with the ACE wrapper:

```python
>>> from delphin.interfaces import ace
>>> ace.compile('../jacy/ace/config.tdl', 'jacy.dat')
[...]
>>> ace.parse('jacy.dat', '犬 が 吠える')
{'SENT': '犬 が 吠える', 'NOTES': ['1 readings, added 183 / 54 edges to chart (22 fully instantiated, 26 actives used, 11 passives used)\tRAM: 730k'], 'WARNINGS': [], 'RESULTS': [{'DERIV': '(267 utterance_rule-decl-finite 4.367251 0 3 (266 head_subj_rule 2.906826 0 3 (263 hf-complement-rule -0.956762 0 2 (262 quantify-n-rule 0.215732 0 1 (10 inu-noun 0.049650 0 1 ("犬" 7 "token [ +FORM \\"犬\\" +FROM \\"0\\" +TO \\"1\\" +ID diff-list [ LIST list LAST list ] +POS pos [ +TAGS null +PRBS null ] +CLASS non_ne [ +INITIAL luk ] +TRAIT token_trait +PRED predsort +CARG \\"犬\\" ]"))) (17 ga 0.150269 1 2 ("が" 8 "token [ +FORM \\"が\\" +FROM \\"2\\" +TO \\"3\\" +ID diff-list [ LIST list LAST list ] +POS pos [ +TAGS null +PRBS null ] +CLASS non_ne [ +INITIAL luk ] +TRAIT token_trait +PRED predsort +CARG \\"が\\" ]"))) (265 unary-vstem-vend-rule 3.336552 2 3 (264 ru-lexeme-infl-rule 2.257695 2 3 (18 hoeru_1 0.000000 2 3 ("吠える" 9 "token [ +FORM \\"吠える\\" +FROM \\"4\\" +TO \\"7\\" +ID diff-list [ LIST list LAST list ] +POS pos [ +TAGS null +PRBS null ] +CLASS non_ne [ +INITIAL luk ] +TRAIT token_trait +PRED predsort +CARG \\"吠える\\" ]"))))))', 'MRS': '[ LTOP: h0 INDEX: e2 [ e TENSE: pres MOOD: indicative PROG: - PERF: - ASPECT: default_aspect PASS: - SF: prop ] RELS: < [ udef_q_rel<0:1> LBL: h4 ARG0: x3 [ x PERS: 3 ] RSTR: h5 BODY: h6 ]  [ "_inu_n_rel"<0:1> LBL: h7 ARG0: x3 ]  [ "_hoeru_v_1_rel"<4:7> LBL: h1 ARG0: e2 ARG1: x3 ] > HCONS: < h0 qeq h1 h5 qeq h7 > ]'}], 'ERRORS': []}
>>> res1 = ace.parse('jacy.dat', '犬 が 吠える')['RESULTS'][0]['MRS']
>>> ace.generate('jacy.dat', res1)
{'WARNING': None, 'ERROR': None, 'SENT': None, 'NOTE': None, 'RESULTS': ['犬 が 吠える']}
```


## Installation and Requirements

pyDelphin is developed for [Python 3](http://python.org/download/)
(3.3+), but it has also been tested to work with Python 2.7. The
[Pygments](http://pygments.org/) package is required for TDL and
SimpleMRS syntax highlighting, and
[NetworkX](http://networkx.github.io/) is required for isomorphism
checking.

pyDelphin does not need to be installed to be used. You can
adjust `PYTHONPATH` to include the pyDelphin directory, or copy the
`delphin/` subdirectory into your project's main directory.

If you want to have the `delphin` packages generally available for
importing, you can use the provided `setup.py` script to install (you
may need to run the following as root):

```bash
$ ./setup.py install
```

The `setup.py` script, unfortunately, does not have an
uninstall/remove option, so in order to remove an installed pyDelphin,
you'll need to track down and remove the files that were installed (in
your Python site packages).

## Sub-packages

The following packages/modules are available:

- `derivation`: [Derivation trees](http://moin.delph-in.net/ItsdbDerivations)
- `itsdb`: [incr tsdb()] profiles
- `mrs`: Minimal Recursion Semantics
- `tdl`: Type-Description Language
- `tfs`: Typed-Feature Structures
- `extra.highlight`: [Pygments](http://pygments.org/)-based syntax
  highlighting (currently just for TDL and SimpleMRS)
- `interfaces.ace`: Python wrapper for common tasks using
  [ACE](http://sweaglesw.org/linguistics/ace/)

## Contributors

- [Michael Wayne Goodman](https://github.com/goodmami/) (primary author)
- [T.J. Trimble](https://github.com/dantiston/) (packaging, derivations, ACE)
