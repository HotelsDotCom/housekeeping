## [4.0.0] - TBD
### Changed
* vacuum-tool: Removed classes for `TablePathResolverTest` to use the ones from [hcommon-hive-metastore](https://github.com/HotelsDotCom/hcommon-hive-metastore).
* vacuum-tool: Removed the configuration for `expected-path-count` as it was not needed.

## [3.1.0] - 2019-04-08
### Fixed 
* Bug where not all DB path entries were looped over in the paging due to deletions.
* Bug in `folder.isEmpty` call for S3AFileSystem that would incorrectly label a folder empty if it had Hadoop `<directoryname>_$folder$` objects.
* Logic for cleaning up paths that were deleted but were not getting deleted from the database and would never be cleaned up.

## [3.0.6] - 2019-02-20
### Changed
* Recursion logic to prevent unnecessary calls.
* The name of the database file for h2 to reflect the name of the schema in the configuration file (instead of having `housekeeping.mv.db` for any schema name).
* Updated `hotels-oss-parent` to 4.0.0 (was 2.3.5).

### Fixed
* Removed default script for creating a housekeeping schema to allow the use of schemas that are already created. See [#111](https://github.com/HotelsDotCom/circus-train/issues/111).
* vacuum-tool: Added the configuration directory to the classpath by default.

## [3.0.5] - 2019-01-23
### Fixed
* NullPointerException when path does not have parent.

## [3.0.4] - 2019-01-15
### Fixed
* Fixed never ending loop in paging query. See [#50](https://github.com/HotelsDotCom/housekeeping/issues/50).

## [3.0.2] - 2019-01-15 [YANKED]
### Fixed
* Fixed deletion of path entries in the database when `pathEventId` is null. See [#48](https://github.com/HotelsDotCom/housekeeping/issues/48).

## [3.0.1] - 2019-01-10 [YANKED]
### Changed
* Refactored housekeeping and vacuum-tool to remove checkstyle and findbugs warnings, which does not impact functionality.
* Upgraded `hotels-oss-parent` to 2.3.5 (was 2.3.3).

## [3.0.0] - 2018-12-12 [YANKED]
### Added
* vacuum-tool: Added validation of configured tables, see housekeeping-vacuum-tool/README.md: tables-validation. 
### Changed
* Database objects are fetched using paging to have an upper limit on the number of them. See [#40](https://github.com/HotelsDotCom/housekeeping/issues/40).
* Added db name and table name columns to housekeeping tables. See [#30](https://github.com/HotelsDotCom/housekeeping/issues/30).
* Changed default max-active connection pool size from 50 to 2 as it was unnecessarily large and having multiple housekeeping jobs was causing RDS issues (too many open connections).

## [2.1.0] - 2018-10-05
### Changed
* Refactored general metastore tunnelling code to leverage hcommon-hive-metastore libraries. See [#23](https://github.com/HotelsDotCom/housekeeping/issues/23).
* Upgraded hotels-oss-parent to 2.3.3 (was 2.0.6).
* Packaging vaccuum-tool results in a jar with a new name: housekeeping-vacuum-tool-all-latest.jar.
* Removed the hcommon-hive-metastore jar from the lib folder and shaded it into housekeeping-vacuum-tool.

## [2.0.0] - 2018-06-22
### Changed
* Refactored module to include the Vacuum Tool (was previously part of [Circus Train](https://github.com/HotelsDotCom/circus-train)). See [#17](https://github.com/HotelsDotCom/housekeeping/issues/17).

## [1.0.4] - 2018-04-25
### Changed
* Default Housekeeping configuration is now provided via `@Bean` `housekeepingEnvironment` only if the bean is not already provided.
### Fixed
* Fixed issue where Housekeeping was failing when database contained a path which no longer exists. See [#18](https://github.com/HotelsDotCom/housekeeping/issues/18).

## [1.0.2] - 2018-02-15
### Added
* Added new housekeeping configuration parameters to set the schema name and the location of the DB initialization script.

## [1.0.1] - 2018-01-15
### Changed
* schema.sql file moved into test resources so that it is not added to classpath of projects that depend upon this project.
* PostConstruct Bean made conditional if Bean implementation is missing.

## [1.0.0] - 2018-01-05
### Added
* Configurable housekeeping entities and repositories.
* Default housekeeping entities and repository.

## [0.0.2] - 2017-12-20
### Changed
* Changed HikariCP version to 2.4.13 Java 7 version for backward compatibility with older JVMs.

## [0.0.1] - 2017-12-12
### Added
* Housekeeping module migrated from [circus-train](https://github.com/HotelsDotCom/circus-train) to this project.
