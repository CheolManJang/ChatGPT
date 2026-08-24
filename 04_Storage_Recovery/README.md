# Storage, Reliability, Backup, and Recovery
> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.
> **기준:** 2026년 8월 24일. 참조 환경: ChatGPT 웹/Work를 사용하며 직접 OpenAI API를 호출하지 않는 개인 ChatGPT Plus 계정. 아키텍처 원칙은 일반적이지만, 관찰된 동작은 테스트한 플랜, 맥락, 권한, 연결된 앱 및 배포 상태에 따라 달라집니다. 중요한 제품 변경 후 다시 검증하십시오.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](../DISCLAIMER.md).
> **사용 시 주의.** 본 자료는 교육 및 일반 정보 제공 목적이며, 어떠한 보증도 제공하지 않습니다. 모든 구현을 직접 평가·테스트하고, 보안과 백업을 확인하며, 필요한 법적 검토를 수행하십시오. [면책 조항](../DISCLAIMER.md)을 참조하십시오.


Documents for source boundaries, connected storage, reliability, backup, and recovery.

- [Storage, Context, and Workspace Boundaries](storage-context-and-workspace-boundaries.md)
- [ChatGPT Reliability and Recovery](chatgpt-reliability-and-recovery.md)
- [Complete Backup and Recovery Manifest](complete-backup-and-recovery-manifest.md)
- [Retired Experiment: Loading Library Rules into ChatGPT Memory](retired-library-to-memory-experiment.md)
- [Multi-Session Synchronization and Locking Lessons](multi-session-synchronization-and-locking.md)

A stored file or backup does not by itself prove complete operational recovery.
