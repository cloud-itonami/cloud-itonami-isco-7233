(ns machineryrepair.advisor
  "RepairAdvisor — the advisor named in this repository's README,
  proposing a machinery-repair operation (approve a repair, approve a
  lift operation, approve hydraulic-pressure work) from a service
  request, equipment history and repair scope. Swappable mock/llm;
  the advisor ONLY proposes — `machineryrepair.governor` checks the
  approved-part membership and test-deviation ceiling independently
  and always escalates lift/hydraulic decisions. Modeled on
  cloud-itonami-isco-4311's advisor.

  A proposal: {:op :approve-repair|:approve-lift-operation|:approve-hydraulic-pressure-work
               :effect :propose :equipment-id str :part str
               :test-deviation-pct number :stake kw :confidence n
               :rationale str}"
  (:require #?(:clj [clojure.edn :as edn] :cljs [cljs.reader :as edn])))

(defprotocol Advisor
  (-advise [advisor store request] "request -> proposal map"))

(defn- infer [_store {:keys [op stake equipment-id part test-deviation-pct] :as request}]
  {:op op
   :effect :propose
   :equipment-id equipment-id
   :part part
   :test-deviation-pct test-deviation-pct
   :stake (or stake :low)
   :confidence (case (or stake :low) :high 0.7 :medium 0.85 :low 0.95)
   :rationale (str "proposed " (name op) " for client " (:client-id request))})

(defn mock-advisor []
  (reify Advisor
    (-advise [_ store request] (infer store request))))

(def ^:private system-prompt
  "You are a machinery-repair advisor. Given a request, propose an
   :op, the :equipment-id, :part and :test-deviation-pct, an honest
   :confidence and a :stake. Never call an unapproved part or an
   over-ceiling test deviation conforming — the governor checks both
   against the registered equipment record. Lift and hydraulic-
   pressure decisions always require human sign-off regardless of
   confidence.")

(defn- parse-proposal [content]
  (try
    (let [p (edn/read-string content)]
      (if (map? p)
        (assoc p :effect :propose)
        {:op :unknown :effect :propose :confidence 0.0 :stake :high
         :rationale "unparseable LLM response"}))
    (catch #?(:clj Exception :cljs js/Error) _
      {:op :unknown :effect :propose :confidence 0.0 :stake :high
       :rationale "LLM response parse failure"})))

(defn llm-advisor
  [chat-model model-generate-fn gen-opts]
  (reify Advisor
    (-advise [_ _store request]
      (let [msgs [{:role :system :content system-prompt}
                  {:role :user :content (str "operation request: " (pr-str request))}]
            resp (model-generate-fn chat-model msgs gen-opts)]
        (parse-proposal (:content resp))))))
