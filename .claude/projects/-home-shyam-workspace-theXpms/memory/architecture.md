# ZenTao Project Architecture Documentation

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Technology Stack](#2-technology-stack)
3. [Directory Structure](#3-directory-structure)
4. [Core Framework](#4-core-framework)
5. [Request Lifecycle](#5-request-lifecycle)
6. [Module Architecture](#6-module-architecture)
7. [Database Layer (DAO)](#7-database-layer-dao)
8. [Frontend Architecture](#8-frontend-architecture)
9. [Zin Declarative UI Framework](#9-zin-declarative-ui-framework)
10. [Database Design](#10-database-design)
11. [Internationalization System](#11-internationalization-system)
12. [Extension System](#12-extension-system)
13. [Testing Framework](#13-testing-framework)
14. [Build System](#14-build-system)
15. [Security Architecture](#15-security-architecture)
16. [Module Catalog](#16-module-catalog)

---

## 1. Project Overview

ZenTao is a comprehensive open-source project management software written in PHP. It covers core processes including product management, project management, quality management, documentation management, organization management, and office management.

- **Version**: 22.0.beta
- **Editions**: open (open source) / biz (business) / max (enterprise)
- **PHP Requirement**: PHP 5.6+, supports up to PHP 8.1+
- **Database**: MySQL/MariaDB (primary), experimental support for DuckDB/OceanBase
- **Character Encoding**: UTF-8

---

## 2. Technology Stack

```
┌─────────────────────────────────────────────────┐
│                 Frontend Layer                   │
│  Traditional: jQuery + PHP Templates + CSS       │
│  Modern: Zin Declarative Framework + ZUI3        │
├─────────────────────────────────────────────────┤
│                Controller Layer                  │
│  framework/base/control.class.php                │
│  module/{name}/control.php                       │
├─────────────────────────────────────────────────┤
│              Business Logic Layer                │
│  model.php → tao.php → zen.php (progressive)    │
├─────────────────────────────────────────────────┤
│              Data Access Layer                   │
│  lib/dao/dao.class.php (chainable query builder) │
├─────────────────────────────────────────────────┤
│                Database Layer                    │
│  MySQL/MariaDB (100+ tables, zt_ prefix)         │
└─────────────────────────────────────────────────┘
```

---

## 3. Directory Structure

```
theXpms/
├── framework/                  # Core framework (MVC base classes)
│   └── base/
│       ├── router.class.php   # Router (3,929 lines)
│       ├── control.class.php  # Base controller (1,258 lines)
│       ├── model.class.php    # Base model (326 lines)
│       └── helper.class.php   # Helper functions
│
├── module/                     # Business modules (104 modules)
│   └── {moduleName}/
│       ├── control.php        # Controller
│       ├── model.php          # Model layer
│       ├── tao.php            # TAO business logic layer
│       ├── zen.php            # ZEN new architecture layer
│       ├── config/            # Module configuration
│       ├── lang/              # Internationalization files
│       ├── view/              # Traditional PHP templates
│       ├── ui/                # Modern Zin declarative pages
│       ├── js/                # JavaScript files
│       ├── css/               # Style files
│       └── test/              # Test files
│
├── lib/                        # Third-party and core libraries
│   ├── dao/                   # Database abstraction layer (16,109 lines)
│   ├── zin/                   # Zin UI framework
│   │   ├── core/              # Core rendering engine
│   │   ├── wg/                # 244 UI components
│   │   ├── zui/               # ZUI3 design system integration
│   │   ├── zentao/            # ZenTao-specific extensions
│   │   └── func.php           # Component factory functions
│   ├── filter/                # Input filtering and validation
│   ├── cache/                 # Caching system
│   └── ...                    # 49 other core libraries
│
├── config/                     # Global configuration
│   ├── config.php             # Default configuration
│   ├── zentaopms.php          # Module mapping and table definitions
│   ├── filter.php             # Input validation rules
│   └── my.php                 # Local overrides (not tracked)
│
├── www/                        # Web entry point
│   ├── index.php              # Main application entry
│   ├── api.php                # RESTful API entry
│   ├── init.php               # Test/CLI initialization
│   └── js/zui3/               # ZUI3 frontend assets
│
├── db/                         # Database schema and migrations
│   ├── zentao.sql             # Main schema (~15,768 lines)
│   └── update*.sql            # Version migration scripts
│
├── extension/                  # Extension system
│   ├── custom/                # Customer customization extensions
│   └── lite/                  # Lightweight edition extensions
│
├── test/                       # Testing framework
│   ├── lib/                   # Core test libraries
│   ├── config/                # Test configuration
│   └── runtime/ztf            # ZTF test runner
│
├── misc/                       # Development tools
│   ├── check.php              # Code checking
│   ├── minifyfront.php        # Frontend minification
│   └── compatibility/         # PHP compatibility checks
│
└── Makefile                    # Build system (452 lines)
```

---

## 4. Core Framework

### 4.1 Router

**File**: `framework/base/router.class.php` (3,929 lines)

The router is the core of the entire application, responsible for request parsing, module loading, and response handling.

**Key Properties**:
```php
class baseRouter
{
    public $config;          // Global configuration object
    public $lang;            // Global language object
    public $dbh;             // PDO database connection
    public $slaveDBH;        // Read replica connection
    public $cache;           // Caching system

    public $moduleName;      // Current module name
    public $methodName;      // Current method name
    public $viewType;        // View type (html/json)
    public $params;          // Request parameters
}
```

**Key Methods**:
- `createApp()` - Create application instance (static factory method)
- `parseRequest()` - Parse URL into module/method/params
- `loadModule()` - Load and execute controller method
- `loadModuleConfig()` - Load module configuration
- `setParams()` - Set request parameters

**URL Format Support**:
```
PATH_INFO:  /user-profile-123          (module-method-param, separator configurable)
GET mode:   /index.php?m=user&f=profile&userId=123
```

### 4.2 Control (Base Controller)

**File**: `framework/base/control.class.php` (1,258 lines)

**Key Properties**:
```php
class baseControl
{
    public $app;             // Application instance (router)
    public $config;          // Global configuration
    public $lang;            // Language object
    public $dbh;             // Database connection
    public $dao;             // DAO query builder
    public $view;            // View data container
    public $moduleName;      // Current module name
    public $methodName;      // Current method name
}
```

**Key Methods**:
- `__construct()` - Initialize global variables, load model, check login
- `loadModel($moduleName)` - Load specified module's model
- `display()` - Render view template
- `send($data)` - Send JSON response
- `locate($url)` - Redirect
- `getCSS()/getJS()` - Load module-level CSS/JS resources

### 4.3 Model (Base Model)

**File**: `framework/base/model.class.php` (326 lines)

**Key Features**:
- Automatically receives references to app, config, lang, dbh, dao global variables
- Can load other module models via `loadModel()`
- Executes database operations through `$this->dao`

---

## 5. Request Lifecycle

```
HTTP Request → www/index.php
    │
    ├─ 1. Load framework classes (router, control, model, helper)
    │
    ├─ 2. router::createApp() → Initialize config, database, language, cache
    │
    ├─ 3. Pre-checks
    │      ├─ checkInstalled() → Redirect to install.php if not installed
    │      └─ checkNeedUpgrade() → Redirect to upgrade.php if upgrade needed
    │
    ├─ 4. parseRequest() → Parse URL to get module/method/params
    │
    ├─ 5. setParams() → Extract and prepare request data
    │
    ├─ 6. Common checks
    │      ├─ checkMaintenance() → Maintenance mode check
    │      ├─ checkPriv() → Permission verification
    │      └─ checkIframe() → iframe handling
    │
    ├─ 7. loadModule() → Load controller class
    │      │
    │      ├─ 7a. Include module/{mod}/control.php
    │      ├─ 7b. Instantiate {Module}Control
    │      │      ├─ Constructor initializes global variables
    │      │      ├─ loadModel() loads model class
    │      │      ├─ Check login status
    │      │      └─ Initialize view object
    │      └─ 7c. Execute $control->{method}($params)
    │             │
    │             ├─ Fetch data through model
    │             ├─ Process business logic
    │             ├─ Populate $this->view
    │             └─ Call display()/send()
    │
    ├─ 8. View Rendering
    │      ├─ Traditional view: Include view/{method}.html.php
    │      ├─ Modern UI: Render ui/{method}.html.php (Zin component tree)
    │      └─ JSON: Encode JSON response
    │
    └─ 9. Output Processing
           ├─ Remove UTF-8 BOM
           ├─ Send response headers
           └─ Output to client
```

---

## 6. Module Architecture

### 6.1 Four-Layer Business Model

ZenTao uses progressive business layering:

```
Control Layer (control.php)
   ↓ calls
Model Layer (model.php) ← Basic business logic + CRUD
   ↑ extends
TAO Layer (tao.php)     ← Complex cross-module business logic
   ↑ extends (optional)
ZEN Layer (zen.php)     ← Latest architecture refactoring
```

### 6.2 Control Layer

**Responsibilities**: HTTP request handling, parameter validation, permission checking, calling models, setting view data

```php
class userControl extends control
{
    public function create()
    {
        if($this->isPost())
        {
            $user = form::data($this->config->user->form->create)->get();
            $this->user->checkBeforeCreateOrEdit($user);
            if(dao::isError()) return $this->send(array('result' => 'fail', 'message' => dao::getError()));

            $userID = $this->user->create($user);
            $this->loadModel('action')->create('user', $userID, 'Created');
            return $this->send(array('result' => 'success', 'id' => $userID));
        }

        $this->view->title = $this->lang->user->create;
        $this->display();
    }
}
```

### 6.3 Model Layer

**Responsibilities**: Basic data operations, business validation, CRUD methods

```php
class userModel extends model
{
    public function getById(int $userID): object|false
    {
        return $this->dao->select('*')
            ->from(TABLE_USER)
            ->where('id')->eq($userID)
            ->andWhere('deleted')->eq('0')
            ->fetch();
    }

    public function checkBeforeCreateOrEdit(object $user): bool
    {
        // Field validation, duplicate detection, password strength check, etc.
    }
}
```

### 6.4 TAO Layer

**Responsibilities**: Extends Model, provides complex cross-module queries and business aggregation

```php
class userTao extends userModel
{
    public function fetchProjects(string $account, string $status = 'all'): array
    {
        return $this->dao->select('t1.role, t2.*')
            ->from(TABLE_TEAM)->alias('t1')
            ->leftJoin(TABLE_PROJECT)->alias('t2')->on('t1.root = t2.id')
            ->where('t1.type')->eq('project')
            ->andWhere('t1.account')->eq($account)
            ->beginIF($status != 'all')->andWhere('t2.status')->eq($status)->fi()
            ->fetchAll('id');
    }
}
```

### 6.5 ZEN Layer

**Responsibilities**: Latest architecture evolution, handles controller-level logic and view property assignment

### 6.6 Inter-Module Interaction

All modules dynamically load other modules via `loadModel()`:

```php
// In control or model
$this->loadModel('action')->create('task', $taskID, 'Created');
$this->loadModel('file')->saveUpload('task', $taskID);
$this->loadModel('user')->updateUserView($products, 'product');
```

### 6.7 Module Configuration System

```
module/{moduleName}/config/
├── config.php          # Base config (required fields, defaults, etc.)
├── form.php            # Form field definitions and validation rules
├── dtable.php          # Data table column definitions
├── search.php          # Advanced search configuration
└── table.php           # Table configuration
```

**form.php example**:
```php
$config->user->form->create = array(
    'account'   => array('required' => true, 'type' => 'string', 'default' => ''),
    'password1' => array('required' => true, 'type' => 'string', 'default' => ''),
    'visions'   => array('required' => true, 'type' => 'array', 'default' => [], 'filter' => 'join'),
);
```

---

## 7. Database Layer (DAO)

### 7.1 Architecture

**File**: `lib/dao/dao.class.php` (~16,109 lines)

```
baseDAO
  └── dao extends baseDAO
```

The DAO provides a chainable query builder, similar to an ORM but more lightweight.

### 7.2 Chainable Query API

```php
// SELECT
$this->dao->select('id, name, email')
    ->from(TABLE_USER)
    ->where('status')->eq('active')
    ->andWhere('type')->in(array('admin', 'pm'))
    ->beginIF($dept)->andWhere('dept')->eq($dept)->fi()
    ->orderBy('id DESC')
    ->limit(0, 20)
    ->page($pager)
    ->fetchAll();      // Returns array of objects
    ->fetch();         // Returns single object
    ->fetchPairs();    // Returns key-value pairs
    ->fetchArray();    // Returns associative array

// INSERT
$this->dao->insert(TABLE_USER)->data($userData)->exec();

// UPDATE
$this->dao->update(TABLE_USER)
    ->set('status')->eq('inactive')
    ->where('id')->eq($userId)
    ->exec();

// DELETE
$this->dao->delete()->from(TABLE_USER)->where('id')->eq($userId)->exec();
```

### 7.3 Conditional Queries

```php
// Conditional chaining
->beginIF($condition)->andWhere('field')->eq($value)->fi()

// Magic methods
$this->dao->findById(1);
$this->dao->findByStatus('active');
```

### 7.4 Error Handling

```php
$this->dao->insert(TABLE_USER)->data($user)->exec();
if(dao::isError()) return dao::getError();
```

### 7.5 Connection Management

- Supports master-slave separation (master for writes, slave for reads)
- PDO persistent connection support
- Query result caching
- Multi-database engine compatibility

---

## 8. Frontend Architecture

### 8.1 Dual-Mode Coexistence

ZenTao supports both traditional view and modern UI frontend modes simultaneously:

| Aspect | Traditional View | Modern UI |
|--------|-----------------|-----------|
| Directory | `module/{mod}/view/` | `module/{mod}/ui/` |
| Syntax | PHP + HTML mixed | Zin declarative components |
| JS File | `{method}.js` | `{method}.ui.js` |
| CSS File | `{method}.css` | `{method}.ui.css` |
| Framework | jQuery + custom | Zin + ZUI3 |
| Events | `$(selector).on()` | `on::event()` |

### 8.2 Traditional View Mode

```php
<?php include '../../common/view/header.html.php';?>
<div id='mainContent' class='main-row'>
  <div class='main-col'>
    <form method='post' action='<?php echo $this->createLink('user', 'create')?>'>
      <table class='table'>
        <tr>
          <th><?php echo $lang->user->account?></th>
          <td><?php echo html::input('account', '', "class='form-control'")?></td>
        </tr>
      </table>
      <?php echo html::submitButton()?>
    </form>
  </div>
</div>
<?php include '../../common/view/footer.html.php';?>
```

### 8.3 Modern UI Mode

```php
<?php
declare(strict_types=1);
namespace zin;

jsVar('roleGroup', $roleGroup);

formPanel(
    set::title($title),
    on::change('input[name=type]', 'changeType'),
    formRow(
        formGroup(
            set::width('1/2'),
            set::label($lang->user->account),
            input(set::name('account'))
        ),
        formGroup(
            set::width('1/2'),
            set::label($lang->user->realname),
            input(set::name('realname'))
        )
    )
);

render();
```

### 8.4 Resource Loading Mechanism

```php
// Automatically loaded in base control:
// 1. module/{mod}/css/common.css or common.ui.css
// 2. module/{mod}/css/{method}.css or {method}.ui.css
// 3. module/{mod}/css/{method}.{lang}.css (language-specific)
// 4. Extension directory CSS
// JS follows the same pattern
```

---

## 9. Zin Declarative UI Framework

### 9.1 Core Architecture

```
lib/zin/
├── core/                     # Core engine
│   ├── node.class.php       # Base node class (component tree)
│   ├── wg.class.php         # Widget component base class
│   ├── h.class.php          # HTML element generator
│   ├── set.class.php        # Property setter
│   ├── props.class.php      # Property management system
│   ├── render.class.php     # Rendering engine
│   └── context.class.php    # Context management
├── wg/                       # 244 UI components
├── zui/                      # ZUI3 design system integration
├── zentao/                   # ZenTao-specific utilities
└── func.php                  # Component factory functions
```

### 9.2 Node System

```php
class node implements JsonSerializable
{
    public string $gid;           // Global unique ID
    public ?node $parent;         // Parent node
    public props $props;          // Property collection
    public array $blocks = [];    // Named content blocks
}
```

### 9.3 Property Setting

```php
set::name('value')              // Set name property
set::className('active')        // Set CSS class
set::url(createLink(...))       // Set URL
on::click('handler')            // Event binding
on::change('selector', 'fn')    // Event with selector
setClass('btn primary')         // CSS class names
setStyle('color', 'red')        // Inline styles
setData('key', 'value')         // data- attributes
```

### 9.4 Core Component Categories

**Form Components**: `formPanel`, `formRow`, `formGroup`, `input`, `textarea`, `picker`, `datePicker`, `radioList`, `checkbox`, `fileInput`

**Data Display**: `dtable`, `datalist`, `entityList`, `statusLabel`

**Layout Components**: `page`, `row`, `col`, `center`, `toolbar`, `featureBar`, `sidebar`, `main`

**Interactive Components**: `btn`, `btnGroup`, `dropdown`, `modal`, `modalTrigger`, `tabs`

**Navigation Components**: `menu`, `nav`, `breadcrumb`, `navbar`, `dropmenu`

### 9.5 Rendering Flow

```
Component Declaration → Node Tree Construction → prebuild() → build() → HTML Generation → Output
```

---

## 10. Database Design

### 10.1 Overview

- **Table Count**: 100+
- **Table Prefix**: `zt_`
- **Engine**: InnoDB
- **Character Set**: UTF-8mb4
- **Naming Convention**: camelCase

### 10.2 Core Table Categories

**Users and Permissions**:
- `zt_user` - User accounts
- `zt_group` - Permission groups
- `zt_usergroup` - User-group associations
- `zt_acl` - Access control lists

**Product Management**:
- `zt_product` - Products
- `zt_branch` - Product branches
- `zt_story` / `zt_storyspec` - User stories/requirements
- `zt_epic` - Business requirements (epics)

**Project Execution**:
- `zt_project` - Projects
- `zt_execution` - Executions (iterations/sprints)
- `zt_task` - Tasks
- `zt_team` - Team members

**Quality Management**:
- `zt_bug` - Bugs/defects
- `zt_case` - Test cases
- `zt_testtask` - Test tasks
- `zt_testplan` - Test plans

**System Support**:
- `zt_action` - Operation logs
- `zt_file` - File attachments
- `zt_config` - Dynamic configuration
- `zt_lang` - Custom translations

### 10.3 Common Field Patterns

```sql
id             INT UNSIGNED AUTO_INCREMENT PRIMARY KEY
deleted        TINYINT DEFAULT 0        -- Soft delete flag
status         VARCHAR(30)              -- Object status
createdBy      VARCHAR(30)              -- Creator account
createdDate    DATETIME                 -- Creation timestamp
editedBy       VARCHAR(30)              -- Last editor
editedDate     DATETIME                 -- Last edit timestamp
```

### 10.4 Table Relationships

ZenTao uses **logical foreign keys** (no physical foreign key constraints):
- `bug.product` → `product.id`
- `task.execution` → `execution.id`
- `story.product` → `product.id`
- `action.objectType` + `action.objectID` → Polymorphic association

---

## 11. Internationalization System

### 11.1 Supported Languages

| Language | Code | Directory |
|----------|------|-----------|
| Simplified Chinese | zh-cn | lang/zh-cn.php |
| Traditional Chinese | zh-tw | lang/zh-tw.php |
| English | en | lang/en.php |
| German | de | lang/de.php |
| French | fr | lang/fr.php |

### 11.2 Language File Structure

```php
// module/user/lang/en.php
$lang->user->common   = 'User';
$lang->user->account  = 'Account';
$lang->user->password = 'Password';
$lang->user->create   = 'Create User';

$lang->user->typeList['inside']  = 'Internal';
$lang->user->typeList['outside'] = 'External';
```

### 11.3 Language Loading Mechanism

```php
$this->app->loadLang('user');       // Load module language
$this->app->loadLang('common');     // Load common language
// Automatically merges extension directory language files
```

---

## 12. Extension System

### 12.1 Extension Directory

```
extension/
├── custom/                   # Customer customization extensions
│   └── {moduleName}/
└── lite/                     # Lightweight edition extensions
    └── {moduleName}/ext/
        ├── config/           # Configuration extensions
        ├── lang/             # Language extensions
        ├── model/            # Model extensions
        ├── control/          # Controller extensions
        ├── view/             # View extensions
        ├── ui/               # UI extensions
        └── css/              # Style extensions
```

### 12.2 Extension Types

1. **Configuration Extensions**: Override module configuration
2. **Language Extensions**: Add/override translations
3. **Model Extensions**: Extend business logic methods
4. **View Hooks**: `.html.hook.php` files inject content during view rendering
5. **Controller Extensions**: Extend controller methods
6. **UI Extensions**: Modern UI page extensions

---

## 13. Testing Framework

### 13.1 Test Architecture

```
test/
├── lib/
│   ├── init.php              # Unit test initialization
│   ├── test.class.php        # baseTest class (reflection-based invocation)
│   ├── ui.php                # UI test initialization
│   ├── yaml.class.php        # YAML data processing
│   └── coverage.php          # Code coverage
├── config/
│   └── config.php            # UI test configuration
├── runtime/
│   └── ztf                   # ZTF test runner
└── spider.php                # UI automation crawler
```

### 13.2 Three-Layer Testing System

```
UI Tests (module/{mod}/test/ui/)
    ↓ Tests user interface interactions
ZEN/TAO Tests (module/{mod}/test/zen/ or tao/)
    ↓ Tests business logic
Model Tests (module/{mod}/test/model/)
    ↓ Tests data access
```

### 13.3 Module Test Directory

```
module/{moduleName}/test/
├── lib/
│   ├── model.class.php       # {moduleName}ModelTest extends baseTest
│   ├── tao.class.php         # {moduleName}TaoTest extends baseTest
│   └── zen.class.php         # {moduleName}ZenTest extends baseTest
├── model/                    # Model layer test scripts
│   ├── yaml/{method}/        # Method-specific test data
│   └── {methodName}.php      # Test script
├── tao/                      # TAO layer test scripts
├── zen/                      # ZEN layer test scripts
├── ui/                       # UI automation tests
│   ├── lib/{action}.ui.class.php
│   ├── page/{action}.php     # Page object definitions
│   └── {action}.php          # UI test script
└── yaml/                     # Common test data
    └── {tableName}.yaml
```

### 13.4 Test Script Format

```php
#!/usr/bin/env php
<?php
/**
title=Test userModel::getById();
timeout=0
cid=0

- Step 1: Query existing user >> admin
- Step 2: Query non-existent user >> 0
- Step 3: Query user with ID 0 >> 0
- Step 4: Query negative ID >> 0
- Step 5: Query non-numeric ID >> 0
*/

include dirname(__FILE__, 5) . '/test/lib/init.php';
include dirname(__FILE__, 2) . '/lib/model.class.php';

// Data preparation
zenData('user')->gen(10);
su('admin');

// Create test instance
$userTest = new userModelTest();

// Execute test steps (must be >= 5)
r($userTest->getByIdTest(1)) && p('account') && e('admin');
r($userTest->getByIdTest(999)) && p() && e('0');
r($userTest->getByIdTest(0)) && p() && e('0');
r($userTest->getByIdTest(-1)) && p() && e('0');
r($userTest->getByIdTest('abc')) && p() && e('0');
```

### 13.5 Test Assertion API

```php
r($result)              // Record test result
p()                     // Check entire return result
p('field')              // Check specific field
p('field1,field2')      // Check multiple fields
p('0:field')            // Check first array element's field
e('expectedValue')      // String expectation
e(123)                  // Numeric expectation
e('0')                  // Falsy value
e('~~')                 // Empty value
su('admin')             // Switch user
```

### 13.6 zendata Test Data Generation

```php
// Direct API
$table = zenData('user');
$table->account->range('admin,user{99}');
$table->status->range('active{50},inactive{50}');
$table->gen(10);

// YAML loading
zendata('user')->loadYaml('user_config', false, 2)->gen(10);
```

### 13.7 Running Tests

```bash
# Syntax check
php module/{mod}/test/{layer}/{method}.php

# Run test
test/runtime/ztf module/{mod}/test/{layer}/{method}.php

# Pass criteria: PASS=1, FAIL=0, SKIP=0
```

---

## 14. Build System

### 14.1 Main Build Targets

```bash
make all           # Full build: clean → ci
make clean         # Clean build artifacts
make common        # Core functionality build
make package       # Create distribution package
make pms           # Standard edition package
make ci            # CI full build
make deb           # Debian package
make rpm           # RedHat package
```

### 14.2 Build Process

1. Copy core files to `zentaopms/` directory
2. Clean unnecessary files (tests, docs, extra language packs)
3. Merge XuanXuan (instant messaging system)
4. Merge database scripts
5. Frontend resource minification (`misc/minifyfront.php`)
6. Create distribution packages (ZIP, TAR.XZ)

### 14.3 Development Tools

```bash
php misc/check.php              # Code checking
php misc/minifyfront.php        # Frontend minification
php misc/difflang.php           # Language file diff
php misc/cn2tw.php              # Simplified to Traditional Chinese conversion
php misc/difftable.php          # Database table diff
```

---

## 15. Security Architecture

### 15.1 Authentication System

- Session-based login management
- Password hashing (bcrypt/md5 compatible)
- Login failure lockout (default: 6 failures locks for 10 minutes)
- IP restriction checking

### 15.2 Permission System

```
common->checkPriv()
   ├─ Check user role (admin, pm, dev, qa, etc.)
   ├─ Check module-method permissions
   ├─ Check workflow permissions
   └─ Allow/deny access
```

### 15.3 Input Filtering

- **Framework level**: `config/filter.php` global filtering rules
- **DAO level**: Prepared statements to prevent SQL injection
- **Form level**: Client-side validation prompts
- **XSS protection**: Output encoding, CSP headers
- **CSRF protection**: Session token verification

---

## 16. Module Catalog

ZenTao contains **104 business modules**, categorized as follows:

### Core Business Modules
| Module | Description |
|--------|-------------|
| product | Product management |
| project | Project management |
| execution | Execution/iteration management |
| story | User story/requirement management |
| task | Task management |
| bug | Bug/defect tracking |
| testcase | Test case management |
| testtask | Test task management |
| build | Build management |
| release | Release management |
| doc | Documentation management |

### Administration Modules
| Module | Description |
|--------|-------------|
| user | User management |
| group | Permission groups |
| company | Organization management |
| dept | Department management |
| admin | System administration |
| custom | Custom configuration |

### Integration Modules
| Module | Description |
|--------|-------------|
| git | Git integration |
| gitlab | GitLab integration |
| gitea | Gitea integration |
| gogs | Gogs integration |
| jenkins | Jenkins CI/CD |
| api | API management |
| webhook | Webhook support |

### Reporting & Analytics
| Module | Description |
|--------|-------------|
| report | Standard reports |
| chart | Charting functionality |
| metric | Metrics calculation |
| bi | Business intelligence |
| screen | Dashboard display |

### Collaboration Modules
| Module | Description |
|--------|-------------|
| todo | To-do items |
| effort | Work hours management |
| attend | Attendance management |
| leave | Leave management |
| im | Instant messaging |
| message | Message notifications |

### Extended Feature Modules
| Module | Description |
|--------|-------------|
| kanban | Kanban board |
| gantt | Gantt chart |
| flow | Workflow engine |
| workflowfield | Workflow fields |
| workflowaction | Workflow actions |
| search | Search |
| tree | Category tree |

---

## Appendix: Key File Index

| File | Lines | Description |
|------|-------|-------------|
| framework/base/router.class.php | 3,929 | Core router |
| framework/base/control.class.php | 1,258 | Base controller |
| framework/base/model.class.php | 326 | Base model |
| lib/dao/dao.class.php | ~16,109 | Database abstraction layer |
| lib/zin/func.php | ~20,675 | Zin component factory functions |
| db/zentao.sql | ~15,768 | Main database schema |
| config/zentaopms.php | ~47,152 | Module mapping configuration |
| Makefile | 452 | Build system |
