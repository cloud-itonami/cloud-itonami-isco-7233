(ns machineryrepair.governor
  "MachineryRepairGovernor — the independent safety/traceability
  layer named in this repository's README/business-model.md, gating
  the robot-dispensed physical work (equipment scanning,
  part-handling) an advisor may propose. The governor never
  dispatches hardware itself. Modeled on cloud-itonami-isco-4311's
  bookkeeping.governor. Repair twist: a proposed replacement part must
  be a member of the registered approved-parts set (no counterfeit or
  unauthorized part), and a proposed post-repair test deviation is
  arithmetic comparison against the registered ceiling.

  HARD invariants (:hard? true, ALWAYS :hold, never overridable):
    1. client provenance — the organization must be registered.
    2. no-actuation      — proposal :effect must be :propose (the
                           governor never dispatches hardware; it only
                           gates what the robot may execute).
    3. equipment basis      — a repair approval must cite REGISTERED
                           equipment belonging to this client.
    4. approved-part membership — the proposed replacement part must
                           be a member of the equipment's registered
                           :approved-parts set (no counterfeit or
                           unauthorized part).
    5. test-deviation ceiling — the proposed post-repair test
                           deviation must not exceed the equipment's
                           registered :max-test-deviation-pct.
  ESCALATION invariants (:escalate? true, ALWAYS human sign-off per
  business-model.md's Trust Controls — these are :high/
  :safety-critical regardless of confidence):
    6. :op :approve-lift-operation (no lift operation without the
                           governor gate).
    7. :op :approve-hydraulic-pressure-work (pressurized-hydraulic
                           work requires human sign-off).
    8. low confidence (< `confidence-floor`)."
  (:require [machineryrepair.store :as store]))

(def confidence-floor 0.6)

(def ^:private always-escalate-ops #{:approve-lift-operation
                                     :approve-hydraulic-pressure-work})

(defn- hard-violations [{:keys [request proposal]} client-record e]
  (let [{:keys [op part test-deviation-pct]} proposal
        repair? (= :approve-repair op)]
    (cond-> []
      (nil? client-record)
      (conj {:rule :no-client :detail "未登録 client"})

      (not= :propose (:effect proposal))
      (conj {:rule :no-actuation :detail "effect は :propose のみ許可（governor はハードウェアを直接起動しない）"})

      (and repair? (nil? e))
      (conj {:rule :unknown-equipment :detail "未登録 equipment への修理承認は不可"})

      (and repair? e (not= (:client-id e) (:client-id request)))
      (conj {:rule :equipment-wrong-client :detail "equipment が別 client のもの"})

      (and repair? e part (not (contains? (:approved-parts e) part)))
      (conj {:rule :unapproved-part
             :detail (str "部品 " part " は登録済み承認集合の外（模造・未承認部品の使用禁止）")})

      (and repair? e (number? test-deviation-pct)
           (> test-deviation-pct (:max-test-deviation-pct e)))
      (conj {:rule :test-deviation-exceeds-ceiling
             :detail (str "試験偏差 " test-deviation-pct "% > 登録済み上限 "
                          (:max-test-deviation-pct e) "%")}))))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `machineryrepair.store/Store`. Pure — never
  mutates the store, never dispatches the robot."
  [request context proposal store]
  (let [client-record (store/client store (:client-id request))
        e (some->> (:equipment-id proposal) (store/equipment store))
        hard (hard-violations {:request request :proposal proposal}
                              client-record e)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        always-risky? (contains? always-escalate-ops (:op proposal))]
    {:ok? (and (not hard?) (not low?) (not always-risky?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? always-risky?))}))
