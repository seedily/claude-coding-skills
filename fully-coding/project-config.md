---
# fully-coding / fully-coding-batch 共享配置。
# 所有 {{config.*}} 占位符必须在此声明；缺失即报错。
# 部署到新项目时，只需修改路径、知识库文档名、框架和错误码配置。

# 路径配置
knowledge_dir: "_knowledge"
output_dir: ".dev-log"

# 知识库文档
requirement_doc: "3.项目需求-*.md"
error_code_doc: "5.服务详细设计（错误码段位分配）.md"
tech_design_glob: "4.技术方案-*.md"
tech_architecture_doc: "4.技术方案-后台系统架构.md"
service_design_glob: "5.服务详细设计-*.md"
feature_impl_glob: "6.功能实现-*.md"
feature_impl_overview: "6.功能实现（概览）.md"
feature_impl_menu_iteration: "6.功能实现（菜单功能迭代）.md"
feature_impl_template: "templates/feature_impl_template.md"
menu_client_apps: ["admin-biz-web", "member-biz-web", "APP"]

# 心跳与续跑
heartbeat_interval_seconds: 300
heartbeat_timeout_seconds: 3600

# 后端框架
framework_type: "grace-ddd"
response_wrapper_success: "DataResponse.success"
response_wrapper_fail: "DataResponse.fail"
base_response_class: "BaseResponse"

# 错误码
error_code_format: "[L][CC][SS][D]"
error_code_utility: "DomainSeq"
error_code_utility_package: "com.grace.demo.domain"
error_code_enums: ["ErrCodeLevel.java", "ErrCodeCategory.java"]
---

# 维护原则

- 本文件是唯一配置来源，不支持运行时覆盖。
- 新增配置占位符时，必须同步在本文件声明对应 key。
- 说明保持简短；详细流程、规则和模板含义写在对应 `rules/`、`agents/`、`templates/` 中。
