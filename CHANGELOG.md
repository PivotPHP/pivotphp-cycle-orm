# Changelog

All notable changes to PivotPHP Cycle ORM will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2025-07-09

### 🚀 **Performance & Compatibility Update**

This release focuses on performance optimizations, cross-platform compatibility, and code quality improvements following the PivotPHP Core v1.1.0 upgrade.

#### 🔄 **Updated**
- **PivotPHP Core**: Updated to v1.1.0 with performance improvements for auto stress scenarios
- **Environment Detection**: Centralized environment detection using `EnvironmentHelper` to reduce code duplication
- **Caching System**: Implemented static caching for environment methods with 7x performance improvement
- **Testing Infrastructure**: Clean PHPUnit exit codes (0) for better CI/CD integration

#### 🌐 **Added**
- **Cross-Platform Support**: Added platform-specific scripts for test coverage
  - `scripts/test-coverage.sh` for Unix/Linux/macOS
  - `scripts/test-coverage.bat` for Windows CMD
  - `scripts/test-coverage.ps1` for PowerShell
  - `scripts/test-coverage.php` for universal PHP-based execution
- **Cache Management**: Added `EnvironmentHelper::clearCache()` method for testing scenarios
- **Performance Benchmarks**: Added benchmark scripts to demonstrate caching benefits

#### 🔧 **Fixed**
- **CI/CD Compatibility**: Fixed PHPUnit exit codes that were causing CI failures
- **Log Pollution**: Suppressed test-specific error logs in testing environment
- **PSR-12 Compliance**: Fixed all code style violations
- **Cross-Platform Issues**: Resolved Windows compatibility issues with inline environment variables

#### 🏗️ **Refactored**
- **Environment Helper**: Centralized cache system using single static array
- **Metrics Collector**: Simplified environment detection using `EnvironmentHelper::isTesting()`
- **Test Configuration**: Optimized PHPUnit configuration for better performance and compatibility
- **Code Organization**: Removed duplicate composer files and obsolete scripts

#### 📊 **Performance Improvements**
- **Environment Detection**: 7x faster performance on subsequent calls with static caching
- **Test Execution**: Reduced test execution time by eliminating unnecessary coverage overhead
- **Memory Usage**: Optimized memory usage by reducing redundant environment checks

#### 🧪 **Testing Enhancements**
- **Clean Exit Codes**: PHPUnit now returns proper exit code 0 for successful test runs
- **CI-Friendly**: Removed confusing error logs from test output
- **Coverage Separation**: Coverage generation is now optional and platform-independent

#### 📚 **Documentation**
- **Cross-Platform Guide**: Updated documentation with platform-specific instructions
- **Performance Notes**: Added performance improvement documentation
- **CLI Usage**: Enhanced CLAUDE.md with better development workflow guidance

#### 🔍 **Technical Details**
- **Compatibility**: Maintains full backward compatibility with existing APIs
- **Dependencies**: Updated to PivotPHP Core v1.1.0 from Packagist
- **Testing**: 67 tests passing with 242 assertions
- **Static Analysis**: PHPStan Level 9 compliance maintained
- **Code Style**: 100% PSR-12 compliant

#### 🎯 **Benefits**
- **Faster Development**: Improved environment detection performance
- **Better CI/CD**: Clean test outputs and exit codes
- **Cross-Platform**: Works seamlessly on Windows, macOS, and Linux
- **Production Ready**: Optimized for high-performance production environments

## [1.0.0] - 2025-07-07

### 🎉 **Initial Stable Release**

First stable release of PivotPHP Cycle ORM integration, providing robust database ORM capabilities for the PivotPHP Framework.

#### Added
- **Cycle ORM Integration**: Complete integration with Cycle ORM for PivotPHP Framework
- **Service Provider**: `CycleServiceProvider` for seamless framework integration
- **Repository Pattern**: Built-in repository pattern support with custom repositories
- **Transaction Middleware**: Automatic transaction handling for requests
- **Entity Validation**: Middleware for entity validation with custom rules
- **Query Monitoring**: Performance monitoring and query logging capabilities
- **Health Checks**: Database health monitoring integration
- **Migration Support**: Schema migration tools and commands
- **Database Factory**: Support for multiple database connections
- **Type Safety**: Full type safety with PHPStan Level 9 compliance

#### Features
- **Multiple Databases**: Support for MySQL, PostgreSQL, SQLite, SQL Server
- **Relationships**: Full support for all Cycle ORM relationship types
- **Migrations**: Schema versioning and migration system
- **Factories**: Entity factories for testing and seeding
- **Events**: Database events and listeners
- **Caching**: Query result caching integration
- **Debugging**: Query debugging and profiling tools
- **Commands**: CLI commands for database operations

#### Technical Details
- **Namespace**: `PivotPHP\CycleORM`
- **Framework**: PivotPHP Core v1.0.0+
- **Cycle ORM**: v2.x compatibility
- **PHP**: 8.1+ with full 8.4 compatibility
- **Standards**: PSR-11, PSR-12 compliant
- **Testing**: Comprehensive test coverage

#### Performance
- **Optimized Queries**: Query optimization and caching
- **Connection Pooling**: Efficient database connection management
- **Lazy Loading**: Intelligent lazy loading of relationships
- **Memory Management**: Optimized memory usage for large datasets

#### Documentation
- Complete integration guide
- API reference documentation
- Performance optimization guide
- Migration from other ORMs
- Best practices and examples

#### CLI Commands
`Commands\EntityCommand`, `MigrateCommand`, `SchemaCommand`, `StatusCommand` — plain
PHP classes (`handle(): int`), not a bundled binary. See README.md's "Custom Commands"
section for the real invocation pattern (no `vendor/bin/pivotphp` is shipped).

#### Basic Usage
```php
use PivotPHP\Core\Core\Application;
use PivotPHP\CycleORM\CycleServiceProvider;

$app = new Application();
$app->register(new CycleServiceProvider($app));

// Use in routes
$app->get('/users', function (CycleRequest $request, $res) {
    $users = $request->repository(User::class)->findAll();
    return $res->json($users);
});
```

### 📋 Release Notes

This initial release provides a complete Cycle ORM integration for PivotPHP Framework, offering:

1. **Database Abstraction**: Work with multiple database systems
2. **Type Safety**: Full type safety and static analysis support
3. **Performance**: Optimized for high-performance applications
4. **Developer Experience**: Rich CLI tools and debugging capabilities
5. **Testing**: Comprehensive test coverage and factories

### 🔄 Future Roadmap

Future releases will focus on:
- Additional database drivers
- Enhanced performance optimizations
- Advanced caching strategies
- Extended CLI tooling
- Community-requested features

### 📞 Support

For questions, issues, or contributions:
- **GitHub**: [https://github.com/PivotPHP/pivotphp-cycle-orm](https://github.com/PivotPHP/pivotphp-cycle-orm)
- **Documentation**: [docs/](docs/)
- **Integration Guide**: [docs/integration-guide.md](docs/integration-guide.md)
- **Examples**: [examples/](examples/)

---

**Current Version**: v1.0.1
**Release Date**: July 9, 2025
**Stability**: Stable
**Framework Requirement**: PivotPHP Core ^1.1.0 (see composer.json)