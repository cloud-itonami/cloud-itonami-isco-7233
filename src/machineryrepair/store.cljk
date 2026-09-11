(ns machineryrepair.store
  "SSoT for the ISCO-08 7233 independent farm & industrial machinery
  repair practice actor (itonami actor pattern, ADR-2607011000 /
  CLAUDE.md Actors section; README's 'Robotics premise' — a
  diagnostic and lift-assist robot performs equipment scanning and
  part-handling under this advisor/governor pair, which never
  dispatches hardware itself). Modeled on cloud-itonami-isco-4311's
  bookkeeping.store.

  Domain:

    client    — a registered organization (:client-id, :name)
    equipment — a registered piece of equipment {:equipment-id
                :client-id :name :max-test-deviation-pct number
                :approved-parts #{part-str}}.
                `:max-test-deviation-pct` is the registered ceiling a
                proposed post-repair test's measured deviation must
                not exceed; `:approved-parts` is the registered set a
                proposed repair's replacement part must be a member
                of (no counterfeit or unauthorized part).
    record    — a committed operating record (approved repair) —
                written ONLY via commit-record!.
    ledger    — append-only audit trail, commit or hold."
  )

(defprotocol Store
  (client [s client-id])
  (equipment [s equipment-id])
  (records-of [s client-id])
  (ledger [s])
  (register-client! [s client])
  (register-equipment! [s e])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (client [_ client-id] (get-in @a [:clients client-id]))
  (equipment [_ equipment-id] (get-in @a [:equipment equipment-id]))
  (records-of [_ client-id] (filter #(= client-id (:client-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-client! [s client]
    (swap! a assoc-in [:clients (:client-id client)] client) s)
  (register-equipment! [s e]
    (swap! a assoc-in [:equipment (:equipment-id e)] e) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:clients {} :equipment {} :records [] :ledger []}
                                   seed)))))
