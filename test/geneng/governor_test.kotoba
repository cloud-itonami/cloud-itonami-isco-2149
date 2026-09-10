(ns geneng.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [geneng.governor :as governor]
            [geneng.store :as store]))

(deftest test-check-unregistered-project
  (testing "check hard-blocks unregistered projects"
    (let [s (store/mem-store)
          req {:project-id "unknown"}
          prop {:op :draft-engineering-analysis :effect :propose :confidence 0.9}
          verdict (governor/check req {} prop s)]
      (is (not (:ok? verdict)))
      (is (:hard? verdict))
      (is (> (count (:violations verdict)) 0)))))

(deftest test-check-non-propose-effect
  (testing "check hard-blocks non-:propose effects"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :draft-engineering-analysis :effect :execute :confidence 0.9}
          verdict (governor/check req {} prop s)]
      (is (not (:ok? verdict)))
      (is (:hard? verdict)))))

(deftest test-check-issue-certified-design
  (testing "check hard-blocks attempts to issue certified designs"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :issue-certified-design :effect :propose :confidence 0.9}
          verdict (governor/check req {} prop s)]
      (is (not (:ok? verdict)))
      (is (:hard? verdict)))))

(deftest test-check-certify-safety-compliance
  (testing "check hard-blocks attempts to certify safety compliance"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :certify-safety-compliance :effect :propose :confidence 0.9}
          verdict (governor/check req {} prop s)]
      (is (not (:ok? verdict)))
      (is (:hard? verdict)))))

(deftest test-check-low-confidence-escalation
  (testing "check escalates on low confidence"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :draft-engineering-analysis :effect :propose :confidence 0.5}
          verdict (governor/check req {} prop s)]
      (is (not (:ok? verdict)))
      (is (:escalate? verdict))
      (is (not (:hard? verdict))))))

(deftest test-check-flag-safety-risk-escalation
  (testing "check escalates on :flag-safety-risk operation"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :flag-safety-risk :effect :propose :confidence 0.9}
          verdict (governor/check req {} prop s)]
      (is (not (:ok? verdict)))
      (is (:escalate? verdict))
      (is (not (:hard? verdict))))))

(deftest test-check-ok-proposal
  (testing "check accepts valid proposals"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :draft-engineering-analysis :effect :propose :confidence 0.8 :stake :low}
          verdict (governor/check req {} prop s)]
      (is (:ok? verdict))
      (is (not (:hard? verdict)))
      (is (not (:escalate? verdict))))))

(deftest test-check-log-project-data
  (testing "check accepts :log-project-data proposals"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :log-project-data :effect :propose :confidence 0.85}
          verdict (governor/check req {} prop s)]
      (is (:ok? verdict))
      (is (not (:hard? verdict)))
      (is (not (:escalate? verdict))))))

(deftest test-check-request-client-review
  (testing "check accepts :request-client-review proposals"
    (let [s (store/mem-store {:projects {"proj-1" {:project-id "proj-1"}}})
          req {:project-id "proj-1"}
          prop {:op :request-client-review :effect :propose :confidence 0.75}
          verdict (governor/check req {} prop s)]
      (is (:ok? verdict))
      (is (not (:hard? verdict)))
      (is (not (:escalate? verdict))))))
