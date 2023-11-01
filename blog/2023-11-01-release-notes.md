---
slug: 2023-11-01-release-notes
title: October 2023
authors: yana
tags: [release-notes, updates]
---

Beda EMR Release Notes. October 2023.

# New Dev Tools for Data Extraction and Data Population Testing

In this Beda EMR release, we introduce a set of powerful development tools aimed at simplifying data extraction and data population testing for Clinical Forms, especially those with complex logic. We understand that automated testing of such logic is crucial for software developers and are committed to providing the tools necessary for this task.

### More details 👇
<!--truncate-->

## Clinical Forms and FHIR Questionnaires

Beda EMR’s Clinical Forms are intricately tied to FHIR Questionnaires, which are used to gather essential medical data. The complexity of these forms, often requiring data pre-population and data extraction, necessitates a robust testing framework.

## TestScript for FHIR Questionnaires

To address this challenge, we leverage the TestScript FHIR resource to create a comprehensive testing framework for Questionnaires. With TestScript, you can now write tests that validate the logic governing data pre-population and extraction within your Clinical Forms.

## Python Interpreter for TestScript

As a significant outcome of this release, we are proud to present a Python interpreter for TestScript. This interpreter can be found on GitHub at [https://github.com/beda-software/testscript-eval-py/](https://github.com/beda-software/testscript-eval-py/). With this tool, you can execute TestScript tests seamlessly within a Python environment, enabling efficient and effective testing of your Clinical Forms.

## Introducing Kaitenzushi — A build system for FHIR Shorthand

Additionally, we are unveiling Kaitenzushi, a tool designed to simplify the process of writing modular TestScripts using FHIR Shorthand. Kaitenzushi can be accessed on GitHub at [https://github.com/beda-software/kaitenzushi/](https://github.com/beda-software/kaitenzushi/). With Kaitenzushi, you can create well-structured, reusable TestScripts, ensuring a more organized and maintainable approach to testing the logic within your Clinical Forms.

We believe that these new tools will revolutionize the way you approach data extraction and data population testing for Clinical Forms. By combining the power of TestScript, FHIR, and Python, we aim to make this process more accessible, efficient, and effective for software developers.

---
Best regards,  
The Beda EMR Team