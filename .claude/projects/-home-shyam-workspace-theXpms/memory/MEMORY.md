# ZenTao Project Memory

## Key Architecture Files
- See [architecture.md](architecture.md) for comprehensive codebase documentation
- Framework core: `framework/base/` (router, control, model)
- DAO: `lib/dao/dao.class.php` (~16K lines)
- Zin UI framework: `lib/zin/` (244 components in `wg/`)
- DB schema: `db/zentao.sql` (~15K lines, 100+ tables with `zt_` prefix)
- Build: `Makefile` (452 lines)

## Module Structure Pattern
Every module follows: `module/{name}/` with control.php, model.php, tao.php, zen.php, config/, lang/, view/, ui/, js/, css/, test/

## Testing
- Unit tests: `test/runtime/ztf module/{mod}/test/{zen|tao|model}/{method}.php`
- Must have >=5 test steps per test case
- zendata for test data generation from YAML configs
- Three test layers: model (data) -> tao (business) -> zen (controller)

## Project Config
- Version: 22.0.beta
- Edition: open source
- Languages: zh-cn, zh-tw, en, de, fr
- CLAUDE.md says reply in Chinese, but user prefers English
