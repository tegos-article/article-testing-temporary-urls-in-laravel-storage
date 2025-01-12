# How to Test Laravel's `Storage::temporaryUrl()`

This repository contains the text version of the article **"How to Test Laravel's `Storage::temporaryUrl()`"**. The
article demonstrates two practical approaches to testing `Storage::temporaryUrl()` in Laravel, providing insights and
solutions to effectively test this feature.

## About the Article

Laravel's `temporaryUrl()` is a powerful method for generating temporary file URLs in storage services like Amazon S3 or
DigitalOcean Spaces. However, testing this method can be challenging due to limitations in `Storage::fake`. This
repository includes the text of the article, offering two approaches to overcome these challenges:

1. Mocking the fake storage driver.
2. Using Laravel's built-in `shouldReceive` method for mocking.

## Repository Content

This repository includes:

- A detailed explanation of the problem with `Storage::temporaryUrl()` in testing.
- Code examples for:
    - The `PriceExport` model.
    - The `PriceExportController` with a `download` method.
    - Two test cases demonstrating different testing approaches.

## How to Access the Full Article

You can read the complete article with all examples and explanations here:  
[How to Test Laravel's `Storage::temporaryUrl()`](https://dev.to/tegos/testing-temporary-urls-in-laravel-storage-20p7)

## Usage

To utilize the examples provided in the repository:

1. Copy the relevant code examples into your Laravel application.
2. Follow the testing approaches outlined to test `Storage::temporaryUrl()` effectively.

---

Feel free to use this content as a guide to strengthen your Laravel application testing practices!
