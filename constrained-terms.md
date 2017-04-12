Constrained Terms
=================

Constrained terms are a term coupled with a FO-formula (provided by
`foform.maude`).

We should make the first part of a constrained term use Stephen Skeirik's
pattern operations when it detects that the term falls in a module where that is
safe. The formulas present are being broken into different categories based on
what we have decision procedures for, so it's only natural that we use the
pattern operations when they are safe.

We also need to extend FOForm to handle terms which fall into CVC4's abilities
directly. This will require infrastructure to determine whether part of a
formula downs correctly into a module.

```{.maude .cterm}
load full-maude.maude
load string.maude
load metafresh.maude
load foform.maude

fmod CTERM-SET is
  protecting FOFORM-DEFINEDOPS + FOFORM-SUBSTITUTIONSET .

  sorts CTerm NeCTermSet CTermSet CTermSet? .
  subsorts Term < CTerm < NeCTermSet < CTermSet < CTermSet? .

  var N : Nat . vars F F' F'' : FOForm . var X : Variable . var 
  var MOD : Module . var S : Substitution . var SS : SubstitutionSet .
  vars T T' : Term . vars CT CT' CT'' : CTerm .
  vars CTS CTS' : CTermSet . vars NeCTS NeCTS' : NeCTermSet .

  op _|_ : Term FOForm -> CTerm [right id: tt prec 52] .
  ------------------------------------------------------
  --- eq T | ff ;; CTS = CTS .

  op _<<_ : CTerm Substitution -> CTerm .
  ---------------------------------------
  eq (T | F) << S = (T << S) | (F << S) .

  op .CTermSet : -> CTermSet .
  op _;;_ : CTermSet CTermSet   -> CTermSet   [ctor assoc comm id: .CTermSet prec 60] .
  op _;;_ : CTermSet CTermSet?  -> CTermSet?  [ctor ditto] .
  op _;;_ : CTermSet NeCTermSet -> NeCTermSet [ctor ditto] .
  ----------------------------------------------------------
  eq NeCTS ;; NeCTS = NeCTS .

  op _[_] : CTermSet? Module -> [CTermSet] [prec 64] .
  ----------------------------------------------------
  eq CTS [ MOD ] = CTS .

  op _++_ : CTermSet? CTermSet? -> CTermSet? [assoc comm id: .CTermSet prec 61] .
  -------------------------------------------------------------------------------
  eq NeCTS ;; CTS ++ NeCTS ;; CTS'  = NeCTS ;; CTS ++ CTS' .
  eq NeCTS        ++ NeCTS' [ MOD ] = NeCTS ;; NeCTS' [owise] .

  ceq T | F ;; CTS ++ CT' ;; CTS' [ MOD ] = T | F'' ;; CTS ++ CTS' [ MOD ] if T' | F' := #varsApart(CT', T | F)
                                                                           /\ S | SS  := #subsumesWith(MOD, T, T')
                                                                           /\ F''     := F \/ (F' /\ #disjSubsts(S | SS)) .

  op _--_ : CTermSet? CTermSet? -> CTermSet? [right id: .CTermSet prec 62] .
  --------------------------------------------------------------------------
  eq .CTermSet    -- NeCTS          = .CTermSet .
  eq NeCTS ;; CTS -- NeCTS ;; CTS'  = CTS -- NeCTS ;; CTS' .
  eq CT ;; NeCTS  -- NeCTS' [ MOD ] = (CT -- NeCTS' [ MOD ]) ;; (NeCTS -- NeCTS' [ MOD ]) .
  eq NeCTS        -- NeCTS' [ MOD ] = NeCTS [owise] . --- Over-approximate when we can't simplify

  --- TODO: Should we use `++` instead? It would mean that by taking the difference, you make some things union-able.
  ceq CT    -- CT' ;; CTS [ MOD ] = .CTermSet if S | SS := #subsumesWith(MOD, CT', #varsApart(CT, CT')) .
  ceq T | F -- CT' ;; CTS [ MOD ] = T | F'' -- CTS [ MOD ] if T' | F' := #varsApart(CT', T | F)
                                                           /\ S | SS  := #subsumesWith(MOD, T, T')
                                                           /\ F''     := F /\ (#disjSubsts(S | SS) => (~ F')) .
  ceq CT    -- CT' ;; CTS [ MOD ] = CT -- (CTS' ;; CTS) [ MOD ] if CTS' := #intersect(CT, CT') .

  op #intersect : CTerm CTerm -> CTermSet? [comm] .
  -------------------------------------------------

  op #varsApart : CTerm CTerm -> CTerm .
  --------------------------------------
  eq #varsApart(CT, CT') = CT .

  op #disjSubsts : SubstitutionSet -> PosEqQFForm? .
  --------------------------------------------------
  eq #disjSubsts(empty)  = ff .
  eq #disjSubsts(S | SS) = #conjSubst(S) \/ #disjSubsts(SS) .

  op #conjSubst : Substitution -> PosEqConj? .
  --------------------------------------------
  eq #conjSubst(none)       = tt .
  eq #conjSubst(X <- T ; S) = X ?= T /\ #conjSubst(S) .

  op #subsumesWith : Module Term Term -> SubstitutionSet .
  op #subsumesWith : Module Term Term Nat -> SubstitutionSet .
  ------------------------------------------------------------
  eq  #subsumesWith(MOD, T, T')    = #subsumesWith(MOD, T, T', 0) .
  ceq #subsumesWith(MOD, T, T', N) = S | #subsumesWith(MOD, T, T', s(N)) if S := metaMatch(MOD, T, T', nil, N) .
  eq  #subsumesWith(MOD, T, T', N) = empty [owise] .
endfm
```