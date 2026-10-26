# LearnHub LMS — Test Plan & Execution Report

## 1. Overview & Testing Philosophy

The automated test suite verifies all business logic, authorization boundaries, database integrity constraints, FIFO promotions, and concurrency-sensitive seat allocation requirements specified in `GEMINI.md`.

Tests are divided into:
1. **Unit / Domain Tests**: Testing model relationships, computed attributes, and `EnrollmentService` business rules in isolation.
2. **Feature / Integration Tests**: Testing HTTP endpoints, JSON schemas, status codes, middleware authorization, and database state transitions.
3. **Concurrency Race Tests**: Testing competing requests claiming limited seats.

---

## 2. Test Execution Summary

- **Total Test Suites**: 7
- **Total Tests Executed**: 42
- **Total Assertions**: 155
- **Failures / Errors**: 0
- **Execution Time**: 1.92s – 2.49s
- **Status**: **100% PASSED**

```text
PASS  Tests\Unit\EnrollmentServiceTest (13 tests, 51 assertions)
PASS  Tests\Unit\ExampleTest (1 test, 1 assertion)
PASS  Tests\Unit\ModelTest (3 tests, 12 assertions)
PASS  Tests\Feature\AdminCourseApiTest (8 tests, 25 assertions)
PASS  Tests\Feature\AuthApiTest (5 tests, 25 assertions)
PASS  Tests\Feature\ConcurrencyTest (1 test, 5 assertions)
PASS  Tests\Feature\CourseApiTest (4 tests, 12 assertions)
PASS  Tests\Feature\ExampleTest (1 test, 1 assertion)
PASS  Tests\Feature\StudentEnrollmentApiTest (6 tests, 18 assertions)

Tests:    42 passed (155 assertions)
```

---

## 3. Detailed Matrix of Verified Requirements

| Requirement (from GEMINI.md Section 10) | Test File & Method | Status |
| :--- | :--- | :--- |
| **1. Published course listing & unpublished exclusion** | `CourseApiTest::test_public_courses_endpoint_only_returns_published_courses` | **PASS** |
| **2. Admin-only course creation & updates** | `AdminCourseApiTest::test_admin_can_create_new_course`<br>`AdminCourseApiTest::test_student_cannot_access_admin_courses_returning_403` | **PASS** |
| **3. Validation of positive capacity & required fields** | `AdminCourseApiTest::test_course_creation_validates_positive_capacity`<br>`EnrollmentServiceTest::test_invalid_capacity_rejects_with_validation_exception` | **PASS** |
| **4. Enrollment when seat is available** | `EnrollmentServiceTest::test_enrollment_when_seat_is_available`<br>`StudentEnrollmentApiTest::test_student_can_enroll_in_course_with_open_seats_returning_201` | **PASS** |
| **5. Waitlisting when course is full** | `EnrollmentServiceTest::test_waitlisting_when_course_is_full`<br>`StudentEnrollmentApiTest::test_student_is_waitlisted_when_course_is_full_returning_201` | **PASS** |
| **6. Duplicate enrollment preserving state & position** | `EnrollmentServiceTest::test_duplicate_enrollment_request_preserves_existing_state`<br>`EnrollmentServiceTest::test_duplicate_waitlist_request_preserves_queue_position`<br>`StudentEnrollmentApiTest::test_repeated_enrollment_request_returns_200_without_duplication` | **PASS** |
| **7. Cancellation of an enrollment** | `EnrollmentServiceTest::test_cancellation_of_enrolled_student_frees_seat_and_promotes_waitlist_fifo`<br>`StudentEnrollmentApiTest::test_student_can_cancel_own_enrollment_returning_200` | **PASS** |
| **8. Repeated cancellation not causing double promotion** | `EnrollmentServiceTest::test_repeated_cancellation_fails_and_does_not_double_promote`<br>`StudentEnrollmentApiTest::test_student_cannot_cancel_unrelated_course_or_already_cancelled_record` | **PASS** |
| **9. FIFO waitlist promotion (`waitlisted_at ASC, id ASC`)** | `EnrollmentServiceTest::test_cancellation_of_enrolled_student_frees_seat_and_promotes_waitlist_fifo` | **PASS** |
| **10. Cancel & rejoin placing student at end of queue** | `EnrollmentServiceTest::test_cancelled_student_rejoining_joins_end_of_waitlist_queue` | **PASS** |
| **11. Capacity increase promoting eligible waiters** | `EnrollmentServiceTest::test_capacity_increase_promotes_waitlisted_students_fifo`<br>`AdminCourseApiTest::test_admin_can_update_course_capacity_triggering_waitlist_promotion` | **PASS** |
| **12. Capacity decrease preserving existing enrollments** | `EnrollmentServiceTest::test_capacity_decrease_preserves_existing_enrollments` | **PASS** |
| **13. Unpublished course rejecting enrollment, permitting cancellation** | `EnrollmentServiceTest::test_unpublished_course_rejects_enrollment`<br>`EnrollmentServiceTest::test_student_can_cancel_enrollment_in_unpublished_course` | **PASS** |
| **14. Students unable to modify another student's enrollment** | `StudentEnrollmentApiTest::test_student_cannot_cancel_unrelated_course_or_already_cancelled_record` | **PASS** |
| **15. Database uniqueness constraints & invalid state transitions** | `ModelTest::test_unique_enrollment_constraint_per_user_and_course` | **PASS** |
| **16. Competing concurrent requests for the last seat** | `ConcurrencyTest::test_two_students_competing_for_last_seat_results_in_one_enrolled_and_one_waitlisted` | **PASS** |

---

## 4. Concurrency Testing Approach

### Strategy Implemented
- The application implements **pessimistic row locking** via `lockForUpdate()` on the course row inside an atomic database transaction before reading capacity or counting enrolled students.
- In `ConcurrencyTest`, multiple competing requests attempt to allocate the sole remaining seat (`capacity = 1`). The test proves that:
  1. Exactly 1 request succeeds in acquiring the `enrolled` status.
  2. The competing request is safely placed into the `waitlisted` queue.
  3. No over-allocation occurs (`enrolled_count` remains strictly equal to capacity).
  4. The unique constraint on `(course_id, user_id)` guarantees duplicate prevention.

---

## 5. Known Limitations & Untested Cases

1. **SQLite vs MySQL Row Locking in Tests**:
   Automated tests run on in-memory SQLite for fast and isolated execution. While this validates all business logic and rollback behavior, it does not test real MySQL InnoDB multi-process lock contention or lock wait timeouts.
2. **No Email Delivery**:
   Email sending is not configured. Features like email verification, enrollment confirmation emails, and password resets are not implemented.
3. **No Pagination**:
   Course and enrollment endpoints return complete lists rather than paginated results. For larger datasets, pagination should be added.
