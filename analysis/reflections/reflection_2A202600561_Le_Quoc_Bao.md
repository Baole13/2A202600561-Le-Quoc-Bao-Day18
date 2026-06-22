# Individual Reflection - Lab 18

**Ten:** Le Quoc Bao  
**Module phu trach:** M1, M2, M3, M4, M5

---

## 1. Dong gop ky thuat

- Module da implement: toan bo 5 module trong `src/` va sua them integration path de pipeline chay duoc tren moi truong hien tai.
- Cac ham/class chinh da viet: `chunk_semantic()`, `chunk_hierarchical()`, `chunk_structure_aware()`, `BM25Search`, `DenseSearch`, `reciprocal_rank_fusion()`, `CrossEncoderReranker.rerank()`, `evaluate_ragas()`, `failure_analysis()`, `summarize_chunk()`, `generate_hypothesis_questions()`, `contextual_prepend()`, `extract_metadata()`, `_enrich_single_call()`.
- So tests pass: `37/37`.

## 2. Kien thuc hoc duoc

- Khai niem moi nhat: parent-child chunking va hybrid retrieval (BM25 + dense + RRF) trong mot pipeline RAG gan voi production.
- Dieu bat ngo nhat: chat luong he thong khong chi phu thuoc vao logic retrieval ma con phu thuoc manh vao metadata version, prompt answer, va do san sang cua moi truong runtime.
- Ket noi voi bai giang: semantic chunking, BM25 + dense fusion, reranking, RAGAS, va contextual enrichment deu map truc tiep vao 5 module cua repo nay.

## 3. Kho khan & Cach giai quyet

- Kho khan lon nhat: pipeline khong the chay "straight line" vi lien tuc gap loi moi truong.
- Cac loi gap phai:
  - `ModuleNotFoundError: No module named 'qdrant_client'`
  - `UnicodeEncodeError: 'charmap' codec can't encode characters...`
  - `OSError: Can't load the model for 'BAAI/bge-m3'`
  - `RAGAS evaluation failed: No module named 'ragas'`
- Cach giai quyet:
  - Cai them dependency con thieu.
  - Chuyen cac log runtime sang ASCII-safe de chay tren Windows console.
  - Them fallback in-memory cho dense search khi Qdrant khong san sang.
  - Them fallback encoder/reranker de tranh bi khoa boi viec tai model nang.
  - Giu M4/M5 co fallback mem khi thieu API key hay package.
- Thoi gian debug: dai hon phan viet unit test; phan lon thoi gian nam o integration va compatibility.

## 4. Neu lam lai

- Se lam khac dieu gi: chuan bi moi truong tu dau, pre-download model, kiem tra disk cache, va khoi dong Qdrant truoc khi code de tranh debug chen vao luc cuoi.
- Module nao muon thu tiep: M4 va M5, vi khi co `ragas` va `OPENAI_API_KEY` thi moi do duoc tac dong that su cua enrichment len retrieval va answer quality.

## 5. Tu danh gia

| Tieu chi | Tu cham (1-5) |
|----------|---------------|
| Hieu bai giang | 4 |
| Code quality | 4 |
| Teamwork | 4 |
| Problem solving | 5 |
