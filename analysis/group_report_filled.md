# Group Report - Lab 18: Production RAG

**Nhom:** Ca nhan  
**Ngay:** 2026-06-22

## Thanh vien & Phan cong

| Ten | Module | Hoan thanh | Tests pass |
|-----|--------|-----------|-----------|
| Le Quoc Bao | M1: Chunking | x | 12/12 |
| Le Quoc Bao | M2: Hybrid Search | x | 5/5 |
| Le Quoc Bao | M3: Reranking | x | 5/5 |
| Le Quoc Bao | M4: Evaluation | x | 4/4 |
| Le Quoc Bao | M5: Enrichment | x | 11/11 |

## Ket qua RAGAS

| Metric | Naive | Production | Delta |
|--------|-------|-----------|-------|
| Faithfulness | 0.0000 | 0.0000 | 0.0000 |
| Answer Relevancy | 0.0000 | 0.0000 | 0.0000 |
| Context Precision | 0.0000 | 0.0000 | 0.0000 |
| Context Recall | 0.0000 | 0.0000 | 0.0000 |

**Luu y:** Diem RAGAS bang 0 vi moi truong chay thieu package `ragas` va khong co `OPENAI_API_KEY`. Tuy nhien, ca `naive_baseline.py` va `src/pipeline.py` deu da chay xong va sinh report JSON.

## Key Findings

1. **Biggest improvement:** Production pipeline da them chunking theo parent-child, hybrid search, rerank fallback, va enrichment fallback; ve kien truc da day du hon baseline ro ret.
2. **Biggest challenge:** Moi truong runtime khong san sang cho production path day du, cu the la Qdrant khong chay, `qdrant-client` ban dau chua duoc cai, `BAAI/bge-m3` qua nang voi dung luong trong, va `ragas` chua co.
3. **Surprise finding:** Phan kho nhat khong phai unit test ma la tinh on dinh khi chay tren Windows va trong moi truong tai nguyen han che; can fallback tot thi pipeline moi chay het vong.

## Presentation Notes (5 phut)

1. RAGAS scores (naive vs production): ca hai report hien la 0.0 do blocker moi truong, khong phai do pipeline khong chay.
2. Biggest win - module nao, tai sao: M2 la win lon nhat vi da bo sung BM25 tieng Viet, dense search, RRF, va fallback in-memory nen pipeline van index/search duoc khi Qdrant khong san sang.
3. Case study - 1 failure, Error Tree walkthrough: cau hoi "Nhan vien duoc nghi bao nhieu ngay phep nam?" de sai do conflict giua `nghi_phep_nam_v2023.md` va `nghi_phep_nam_v2024.md`.
4. Next optimization neu co them 1 gio: cai `ragas`, cap `OPENAI_API_KEY`, khoi dong Qdrant, gan metadata version/current, va toi uu answer prompt cho policy co nhieu dieu kien.
