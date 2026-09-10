(ns geneng.governor
  "GenengGovernor — the independent safety/traceability layer for
  the ISCO-08 2149 general engineering professionals actor. Wired as its own
  `:govern` node in `geneng.actor`'s StateGraph, downstream of
  `:advise` — the Advisor has no notion of project provenance,
  professional responsibility, or the licensed engineer's exclusive domain,
  so this MUST be a separate system able to reject a proposal (itonami actor
  pattern, per ADR-2607011000 / CLAUDE.md Actors section).

  This actor is a support role: it drafts/prepares engineering analysis
  material for a licensed engineer's own review and professional sign-off,
  in an engineering specialty not covered by the fleet's other more specific
  engineering actors. It NEVER itself issues a final certified engineering
  design or safety certification — those remain the licensed engineer's
  exclusive professional responsibility.

  `check` is a pure function of (request, context, proposal, store) ->
  verdict; it never mutates the store. The StateGraph's `:decide` node
  routes on the verdict:
    :hard? true                → :hold  (irreversible, no write)
    :escalate? true            → :request-approval (interrupt-before)
    otherwise                  → :commit

  HARD invariants (:hard? true, ALWAYS :hold, never overridable):
    1. project provenance  — the request's project must be registered.
    2. no-actuation        — proposal :effect must be :propose.
    3. no-certification    — any attempt to issue a final certified
       engineering design or safety certification is a permanent
       block (that is the licensed engineer's exclusive
       professional responsibility).

  ESCALATION invariants (:escalate? true, ALWAYS human sign-off):
    4. :op :flag-safety-risk (always escalates).
    5. low confidence (< `confidence-floor`)."
  (:require [geneng.store :as store]))

(def confidence-floor 0.6)
(def ^:private escalating-ops #{:flag-safety-risk})

(defn- hard-violations [{:keys [proposal]} project-record]
  (cond-> []
    (nil? project-record)
    (conj {:rule :no-project :detail "未登録 project"})

    (not= :propose (:effect proposal))
    (conj {:rule :no-actuation :detail "effect は :propose のみ許可（直接書込禁止）"})

    (or (= :issue-certified-design (:op proposal))
        (= :certify-safety-compliance (:op proposal)))
    (conj {:rule :no-certification :detail "certified engineering designs and safety certifications are licensed engineer's exclusive responsibility"})))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `geneng.store/Store`. Returns
  `{:ok? bool :violations [...] :confidence n :hard? bool :escalate? bool}`."
  [request context proposal store]
  (let [project-record (store/project store (:project-id request))
        hard (hard-violations {:proposal proposal} project-record)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        escalating-op? (contains? escalating-ops (:op proposal))]
    {:ok? (and (not hard?) (not low?) (not escalating-op?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? escalating-op?))}))
