## 生产中的postgres性能调整
如果你在生产环境中运行 Postgres，那么你将会遇到查询缓慢、表锁、无限等待迁移和错误的问题。如果不是这样，那么对你有好处，你又是怎么做到的？这并不意味着 Postgres 不再是正确工具，而是意味着你需要揭开帷幕，看看下面发生了什么。
截至目前，我发现的最好工具是 pgbadger 。你可以用它解决几乎所有的 Postgres 问题。
这是一个 Perl 命令行工具，它将 Postgres（如果你使用 AWS 的话，就是 RDS）日志作为输入，并输出报告。该报告的好坏取决于你在 Postgres 上启用的日志。因此，在第一步时你可能需要启用这些日志：
```
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0
log_autovacuum_min_duration = 0
log_error_verbosity = default
log_min_duration_statement = 1s
```
此外，你可能还需要启用 pg_stat_statements 语句来实时分析查询，并启用 auto_explain 来自动解释日志中运行缓慢的查询。
运行报告：
```
pgbadger --prefix '%m %u@%d %p %r %a : ' /pglog/postgresql.log
```
该报告将汇总数据，并提供有关 Postgres 所做工作的大量信息。你可以找到关于错误、最慢查询、等待最长查询、获取的锁类型、临时文件是否用于排序、检查点运行的频率、真空运行的频率以及其他类似信息。有了这些数据，你就可以识别和修复运行缓慢的查询，并通过调优提高 Postgres 的性能。
你可以持续运行此报告（CLI 支持增量模式），从而随时掌握新问题。
另外，如果你想理解解释输出，可以用https://tatiyants.com/的pev这款工具。
该工具将解释 JSON 和原始查询作为输入，并将在可视化树形图中对解释输出进行解释：
正如你所见，节点将有最大、最慢、最贵等标签。这将帮助你根据 Postgres 的执行方式来优化查询。

