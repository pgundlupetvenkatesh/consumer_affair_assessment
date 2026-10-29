# Weaknesses
1. There is no Contact form in the shared target url and the test waits for 30s for the contact form to load. The test never checks if it has reached the right target page before filling field values. It fails with a misleading timeout instead of a clear ‘wrong page’ error.
2. Checking for ‘thank’ keyword in the whole raw HTML in the page. Hidden elements, scripts, meta tags or unrelated text containing that word can cause a false pass.
3. The selectors depend on the order of elements on the page. If the UI is restructured, they may silently fill or click the wrong elements, sending bad data or producing a false pass.
4. Fixed sleeps make the test flaky on slow runs and slow on fast ones; use auto-waiting or expect() assertions instead.
5. If test fails, browser.close() will never run leaving browser processes running that consume CI resources. Use `pytest-playwright` page fixture to handle clean setup and cleanup.

# My Complete Prompt to Claude
You are a senior SDET and test automation architect. Your task is to review and improve the provided Python/Playwright end-to-end test for a customer-facing lead submission form. You have access to the application's source code and browser automation environment, but no existing testing framework or prior context.
Your goal is to produce reliable, maintainable, production-quality automation and demonstrate that it detects broken customer journeys rather than merely passing against a working website.
1. **Inspect the application before coding**
Start by inspecting the repository, application source code, routes, existing dependencies, and available tests. Launch the application and inspect the target URL using the browser automation environment.
The original test navigates to https://example.com/contact, but this URL may not contain the expected contact form. Verify the actual application URL and identify the correct lead submission journey from the source code and browser.
Before interacting with the form:
   * Verify that the browser has reached the expected page using a meaningful page identifier, such as a page heading, URL, or form-specific accessible locator.
   * If the expected page or form is missing, fail early with a clear, actionable error rather than waiting for a field interaction to time out.
   * Do not invent routes, field names, selectors, or expected messages.
   * Inspect the actual form fields, validation rules, submission mechanism, and success and failure states.
2. **Replace brittle selectors**
The original test uses positional selectors such as page.locator("input").nth(0) and page.locator("button").last.
Replace these with reliable, accessible locators based on the actual application, preferring:
   * get_by_role()
   * get_by_label()
   * get_by_placeholder() where appropriate
   * Stable test IDs when justified and supported by the application. 

   Verify that each locator identifies the intended element. Avoid selectors that depend on DOM order, generated class names, or incidental page structure. If the application does not expose reliable locators, document the issue and recommend the smallest appropriate improvement.
3. **Eliminate weak assertions and fixed sleeps**
Remove time.sleep() and replace it with Playwright's built-in auto-waiting and web-first assertions using expect().
Replace assert "thank" in page.content().lower() with an assertion against a specific, visible success confirmation associated with the current submission.
Ensure the test cannot pass because the word "thank" appears in hidden elements, scripts, metadata, unrelated text, or stale page content.
Verify meaningful outcomes, including:
   * The correct page and form are loaded.
   * Valid input is accepted.
   * The submission produces the expected success confirmation.
   * Invalid input is rejected with appropriate feedback.
   * Submission failures do not produce a false success result.
   
   Use the application's actual behavior to determine expected outcomes. Avoid arbitrary waits, unnecessary timeout increases, and assertions against undocumented implementation details.
4. **Improve browser lifecycle and test isolation**
Replace manual browser lifecycle management with pytest and the official Playwright pytest integration where appropriate.
   * Use the page fixture provided by pytest-playwright so browser and page resources are managed through fixture setup and teardown.
   * Do not manually launch and close the browser inside each test when the fixture already manages that lifecycle.
   * Ensure each test has an isolated browser context and does not depend on another test's state.
   * Configure the base URL through an environment variable with a documented default appropriate to the application.
   * Keep dependencies minimal and configuration reproducible.
   * Capture useful screenshots and Playwright traces on failure.
   * Never hardcode credentials, use real customer data, or expose secrets in logs.
   
   Follow existing repository conventions when they are sound. Do not introduce unnecessary abstractions or modify unrelated application files.
5. **Add meaningful test coverage**
Create a focused pytest suite for the actual lead submission journey.
At minimum, cover:
   1. Successful submission of valid lead information.
   2. Missing required fields.
   3. Invalid input, such as a malformed email address if the form includes an email field.
   4. Appropriate failure handling when the submission cannot be completed, where this can be tested reliably.
   
   Use unique test data where necessary. Verify the submission through authoritative evidence available in the test environment, such as a verified backend response or supported test API, when applicable. Do not assume that a success message alone proves backend persistence.
   Ensure invalid submissions are rejected and do not create a lead. Keep tests independent and use parameterization for related validation cases when it improves maintainability.
6. **Demonstrate that the test detects broken behavior**
This is a mandatory requirement. Do not stop after obtaining a passing test run against the working application.
First, run the suite against the working application and record the baseline results.
Introduce a controlled failure that prevents the lead from being accepted, and rerun the same positive-path test. Demonstrate that it fails specifically because the expected successful submission outcome did not occur. Then restore normal behavior and demonstrate that the same test passes again. Do not count a setup failure or unrelated timeout as successful regression detection.
Then introduce a controlled, temporary failure in a safe test environment to break the lead submission journey. Prefer using Playwright routing or a test fixture to simulate a failed submission request, or use a supported test backend that rejects or discards the submission. If necessary, use a reversible mutation in a disposable environment.
Verify that:
   * The positive-path test fails when a valid lead cannot actually be submitted.
   * The failure is caused by the expected missing success outcome or failed submission, not by an unrelated timeout or broken test setup.
   * The test does not pass because of stale content, an unrelated success message, or an incorrectly matched element.
   
   Where feasible, also introduce a controlled mutation that bypasses required-field validation and demonstrate that the corresponding negative test detects the regression.
   After each experiment, restore the original behavior and rerun the normal suite to confirm that it passes again.
   Keep failure-injection logic separate from normal production-path tests. Do not modify production data or leave temporary routes, mutations, or test hooks behind.
   If the environment prevents a mutation experiment, explain the limitation and perform the closest feasible alternative. Never claim an experiment passed or failed without executing it and observing the result.
7. **Execute and review the implementation**
Work iteratively:
   1. Inspect the repository and verify the actual target page.
   2. Document the test scenarios and expected observable outcomes.
   3. Implement the test suite, fixtures, configuration, and documentation.
   4. Run the tests against the working application.
   5. Execute the controlled failure experiment and verify that the relevant test fails for the correct reason.
   6. Restore the working application and rerun the baseline suite.
   7. Review the final diff for brittle selectors, arbitrary sleeps, weak assertions, resource leaks, test-order dependencies, and accidental application changes.
   
   Use idiomatic Python, descriptive test names, appropriate type hints, and PEP 8 conventions. Do not weaken assertions or suppress exceptions to make tests pass.
8. **Definition of done and final report**
The solution is complete only when the target page is verified before form interaction, selectors are reliable, assertions are specific and meaningful, fixed sleeps are eliminated, browser resources are managed through fixtures, and the tests detect a deliberately broken submission journey.
Provide a concise final report containing:
   * Files created or modified.
   * Exact setup and test execution commands.
   * Actual baseline test results, including passed, failed, and skipped counts where available.
   * The failure-injection experiment, the expected result, and the observed result.
   * Confirmation that the application was restored and the baseline tests passed again.
   * Any remaining limitations or unverified behavior.
   
   Begin by inspecting the actual application and repository. Prioritize early failure with actionable diagnostics, reliable assertions, deterministic execution, proper resource management, and evidence that the tests can detect regressions.