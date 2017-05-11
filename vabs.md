```{.maude .vabs}
load universal-construction.maude
load foform.maude
load ../src/intersection.maude
load ../tests/test-modules.maude

fmod VARIABLE-ABSTRACTION is

    extending MODULE-EXPRESSION .
    extending FOFORM .

    op VABS : -> ModUniversalConstruction .

    vars M1 M2 : Qid .

    vars V F T T' TL TL' NV : Variable .

    ceq VABS < M1 , M2 >
      = exists ( pr 'FOFORMSIMPLIFY-IMPL .
                 pr M1 .
                 pr M2 .
                 pr 'FVAR-CONCRETE . )
               ( op 'vabs : 'EqConj -> 'EqConj [none] . )
               ( eq 'vabs['_?=_[T, T']] = '_?=_[T, T'] [owise] .

                ceq 'vabs['_?=_[T, T']] = '_/\_['vabs['_?=_[T, NV]],'vabs['_?=_[T', NV]]] 
                 if 'not_['_or_['_::`Variable[T], '_::`Variable[T']]] = 'true.Bool
                 /\ NV := 'joint-variable['upModule[upTerm(M1), 'false.Bool], 'upModule[upTerm(M2), 'false.Bool], T] [none] .

                ceq 'vabs['_!=_[T, T']] = '_/\_['vabs['_?=_[T, NV]],'vabs['_!=_[T', NV]]] 
                 if 'not_['_or_['_::`Variable[T], '_::`Variable[T']]] = 'true.Bool
                 /\ NV := 'joint-variable['upModule[upTerm(M1), 'false.Bool], 'upModule[upTerm(M2), 'false.Bool], T] 
                 /\ 'sortLeq['upModule[upTerm(M1), 'false.Bool], 'leastSort['upModule[upTerm(M1), 'false.Bool], T], 'leastSort['upModule[upTerm(M2), 'false.Bool], NV]] = 'true.Bool [none] .

                ceq 'vabs['_?=_[V, '_`[_`][F, '_`,_[TL, T, TL']]]]
                  = '_/\_['vabs['_?=_[T, NV]], 'vabs['_?=_[V, '_`[_`][F,'_`,_[TL, NV, TL']]]]]
                 if 'not_['_::`Variable[T]] = 'true.Bool 
                 /\ NV := 'joint-variable['upModule[upTerm(M1), 'false.Bool], 'upModule[upTerm(M2), 'false.Bool], T] [none] .

                ceq 'vabs['_!=_[V, '_`[_`][F, '_`,_[TL, T, TL']]]]
                  = '_/\_['vabs['_?=_[T, NV]], 'vabs['_!=_[V, '_`[_`][F,'_`,_[TL, NV, TL']]]]] 
                 if 'not_['_::`Variable[T]] = 'true.Bool 
                 /\ NV := 'joint-variable['upModule[upTerm(M1), 'false.Bool], 'upModule[upTerm(M2), 'false.Bool], T] [none] . )
        
        if V := var('V, 'Variable)
        /\ F := var('F, 'Qid)
        /\ T := var('T, 'Term)
        /\ T' := var('T', 'Term)
        /\ TL := var('TL, 'TermList)
        /\ TL' := var('TL', 'TermList)
        /\ NV := var('NV, 'Variable) .
endfm

--- VARIABLE ABSTRACTION
--- Input: Conjunction of Equations of the form T ?= T' and T != T'

fmod VABS is
    pr FOFORM .
    pr FOFORMSIMPLIFY-IMPL .
    pr TEST-MODULE .
    pr FVAR-CONCRETE .

    vars V NV : Variable .
    var F : Qid .
    vars T T' : Term .
    vars TL TL' : TermList .
    vars M1 M2 : ModuleExpression .
    var EqA : EqAtom .
    var EqC : EqConj .

    op destruct : ModuleExpression ModuleExpression EqConj -> EqConj .
    op vabs :     ModuleExpression ModuleExpression EqConj -> EqConj .

    ceq destruct(M1, M2, T ?= T') = T ?= NV /\ T' ?= NV
      if not (T :: Variable or T' :: Variable)
      /\ NV := joint-variable(M1, M2, T) .

    ceq destruct(M1, M2, T != T') = T ?= NV /\ T' != NV
      if not (T :: Variable or T' :: Variable)
      /\ NV := joint-variable(M1, M2, T) 
      /\ sortLeq(M1, leastSort(M1, T), leastSort(M1, NV)) = true .

    ceq destruct(M1, M2, T != T') = T ?= NV /\ T' != NV
      if not (T :: Variable or T' :: Variable)
      /\ NV := joint-variable(M1, M2, T) 
      /\ sortLeq(M2, leastSort(M2, T), leastSort(M2, NV)) = true .

    ceq vabs(M1, M2, F[TL, T, TL'] ?= V)
      = vabs(M1, M2, F[TL, NV, TL'] ?= V) /\ vabs(M1, M2, NV ?= T)
      if not (T :: Variable) 
      /\ NV := joint-variable(M1, M2, T) .

    ceq vabs(M1, M2, F[TL, T, TL'] != V)
      = vabs(M1, M2, F[TL, NV, TL'] != V) /\ vabs(M1, M2, NV ?= T)
      if not (T :: Variable) 
      /\ NV := joint-variable(M1, M2, T) .

    eq vabs(M1, M2, EqA /\ EqC) = vabs(M1, M2, destruct(M1, M2, EqA)) /\ vabs(M1, M2, EqC) .
    eq vabs(M1, M2, EqA) = EqA [owise] .
endfm
    
reduce upModule('VABS, false) .
reduce vabs('TEST-MODULE, 'TEST-MODULE, 'x:A ?= 'f['a.A]) .
    
reduce in VARIABLE-ABSTRACTION : resolveNames(#upModule('TEST-MODULE deriving VABS < 'TEST-MODULE, 'TEST-MODULE >)) .
reduce in VARIABLE-ABSTRACTION : wellFormed(resolveNames(#upModule('TEST-MODULE deriving VABS < 'TEST-MODULE, 'TEST-MODULE >))) .
reduce downTerm(getTerm(metaReduce( resolveNames(#upModule('TEST-MODULE deriving VABS < 'TEST-MODULE, 'TEST-MODULE >))
                                  , 'vabs[upTerm('x:A ?= 'f['a.A])])), ff) .
reduce downTerm(getTerm(metaReduce( resolveNames(#upModule('TEST-MODULE deriving VABS < 'TEST-MODULE, 'TEST-MODULE >))
                                  , 'vabs[upTerm('f['f['a.A]] ?= 'f['a.A])])), ff) .
*** reduce downTerm(getTerm(metaReduce( resolveNames(#upModule('TEST-MODULE deriving VABS < 'TEST-MODULE >))
***                                   , 'vabs['_/\_[upTerm('f['f['a.A]] ?= 'f['a.A]), upTerm('f['f['a.A]] ?= 'f['a.A])]])), ff) .
```