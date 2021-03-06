---
description: Basic voting process
---

# Voting process

This archetype defines a basic voting process. The chairman is responsible for registering voters. The use of the `mutable` extension gives the chairman the extra right to modify the voting agenda, in Created state only.

The result of the vote is computed with the `bury` action: winners are ballots with the highest number of votes.

{% code title="voting\_process.arl" %}
```ocaml
archetype voting_process

variable[%transferable%] chairperson : role = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

(* vote start *)
variable[%mutable (chairperson, (instate (Created)))%] startDate : date = 2019-11-12T00:00:00

(* vote deadline *)
variable[%mutable (chairperson, (instate (Created)))%] deadline : date = 2020-11-12T00:00:00

asset voter identified by addr {
  addr : role;
  hasVoted : bool
}

asset ballot identified by value {
  value   : string;
  nbvotes : int;
}

asset winner {
  winvalue : string
}

(* state machine *)
states =
 | Created initial with { s1 : winner.isempty(); }
 | Voting          with { s2 : winner.isempty(); }
 | Buried

action register (v : role) {
  called by chairperson
  require {
    c1 : state = Created;
  }
  effect {
    voter.add({ addr = v; hasVoted = false })
  }
}

transition start () {
  from Created
  to Voting when { now > startDate }
}

action vote (val : pkey of ballot) {
   require {
     c2 : voter.contains(caller);
     c3 : state = Voting;
     c4 : voter.get(caller).hasVoted = false;
   }

   effect {
     voter.update (caller, { hasVoted = true });
     if ballot.contains(val) then
      ballot.update(val,{ nbvotes += 1})
     else
      ballot.add({ value = val; nbvotes = 1})
   }
}

transition bury () {
  require {
    c5 : now > deadline;
  }
  from Voting
  to Buried
  with effect {
    let nbvotesMax = ballot.max(the.nbvotes) in
    for b in ballot do
      if (b.nbvotes = nbvotesMax)
      then winner.add({ winvalue = b.value })
    done
  }
}

specification {
  contract invariant s3 {
    startDate < deadline
  }
  contract invariant s4 {
    (voter.select(the.hasVoted = true)).count() = ballot.sum(the.nbvotes)
  }
  contract invariant s5 {
    forall w in winner,
           forall b in ballot,
             let some wb = ballot.get(w.winvalue) in
             b.nbvotes <= wb.nbvotes
             otherwise false
  }
}

```
{% endcode %}

