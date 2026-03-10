# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

### Added

- v0.1.0 Merge UNSW-NB15 datasets to create a CSV file which contains all network data with suitable column headers.
- v0.2.0 Check the shape, feature names, dtypes, statistics, missing value counts, infinite value counts and class distribution (binary label + attack categories) of DataFrame.
- v0.3.0 Drop redundant features, strip whitespace, handle missing values, remove duplicates and impossible records.

### To Add

- Data Cleaning
- Exploratory Data Analysis (EDA)
- Feature Engineering
- Predictive Modelling and Model Evaluation
- Data Visualisation (Confusion Matrix)

## 0.3.2 - 10/03/2026

### Changed

- Set max column width for Pandas DataFrames

## 0.3.1 - 10/03/2026

### Changed

- Include '-' as missing value in isna().

## 0.3.0 - 03/03/2026

### Added

- Drop redundant features, strip whitespace, handle missing values, remove duplicates and impossible records.

## 0.2.1 - 23/02/2026

### Fixed

- FileNotFoundError when loading raw UNSW-NB15 files due to directory restructure.

## 0.2.0 - 23/02/2026

### Added

- Check the shape, feature names, dtypes, statistics, missing value counts, infinite value counts and class distribution (binary label + attack categories) of DataFrame.

## 0.1.0 - 22/02/2026

### Added

- Merge UNSW-NB15 datasets to create a CSV file which contains all network data with suitable column headers.