# ECHO-PPI migration summary (Cursor)

Completed migration to **ECHO-PPI** manuscript identity with new manuscript `manuscript/echo_ppi_main.tex` / `echo_ppi_main.pdf` (**32 pages**), 10 figures, 17 tables, integrated appendices, and honest Gavin benchmarks.

**Reproduce:** `cd cosmos_ppi && bash scripts/reproduce_echo_ppi_final.sh`

**Key result:** F1 0.166 (matches MCL+overlap); bundle completeness ~0.99; does not outperform overlap heuristic on F1.

See `reports/echo_ppi_final_report.md` for full file list and quality checks.
