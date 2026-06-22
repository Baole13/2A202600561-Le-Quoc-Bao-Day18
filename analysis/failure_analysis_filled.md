# Failure Analysis - Lab 18: Production RAG

**Nhom:** Ca nhan  
**Thanh vien:** Le Quoc Bao -> M1, M2, M3, M4, M5

---

## RAGAS Scores

Ghi chu trung thuc: `src/pipeline.py` da chay end-to-end va sinh `ragas_report.json`, nhung moi truong hien tai chua co package `ragas` va khong co `OPENAI_API_KEY`, nen diem RAGAS deu fallback ve `0.0`. Vi vay bang duoi day phan anh trang thai moi truong khi chay lab, khong phai chat luong retrieval thuc te.

| Metric | Naive Baseline | Production | Delta |
|--------|---------------|------------|-------|
| Faithfulness | 0.0000 | 0.0000 | 0.0000 |
| Answer Relevancy | 0.0000 | 0.0000 | 0.0000 |
| Context Precision | 0.0000 | 0.0000 | 0.0000 |
| Context Recall | 0.0000 | 0.0000 | 0.0000 |

## Runtime Summary

- `pytest` da pass `37/37` tests.
- `python -u src/pipeline.py` da chay het 20 cau hoi va luu `ragas_report.json`.
- `python -u naive_baseline.py` da chay va luu `naive_baseline_report.json`.
- Qdrant local khong san sang, nen M2 dense search da fallback sang in-memory vector search.
- Model `BAAI/bge-m3` khong load duoc do thieu dung luong cache Hugging Face, nen M2 da fallback sang encoder nhe hon/heuristic.
- `ragas` chua duoc cai dat va `OPENAI_API_KEY` de trong, nen M4 khong tinh duoc 4 metric that su.

## Bottom-5 Failure Candidates

Do khong co per-question RAGAS output, 5 muc duoi day la failure candidates co xac suat cao nhat duoc suy ra tu corpus, test set, va hanh vi retrieval quan sat trong luc chay pipeline.

### #1
- **Question:** Nhan vien duoc nghi bao nhieu ngay phep nam?
- **Expected:** Chinh sach hien hanh `nghi_phep_nam_v2024.md` la 15 ngay; ban 2023 chi con gia tri tham khao.
- **Got:** De bi keo nham ve 12 ngay neu retrieval trung `nghi_phep_nam_v2023.md`.
- **Worst metric:** Context Precision / Context Recall
- **Error Tree:** Output sai -> Context dung? -> Co the khong, vi co 2 version chinh sach cung noi ve cung mot chu de.
- **Root cause:** Version ambiguity. Corpus co tai lieu cu va moi, nhung pipeline chua co metadata filter theo version/hieu luc.
- **Suggested fix:** Them metadata `version`, `effective_date`, `is_current`; uu tien tai lieu moi trong rerank hoac sau retrieval.

### #2
- **Question:** Mat khau phai co toi thieu bao nhieu ky tu?
- **Expected:** Ban hien hanh `mat_khau_v2.md` yeu cau 12 ky tu.
- **Got:** De tra loi 8 ky tu neu chunk tu `mat_khau_v1.md` duoc xep cao.
- **Worst metric:** Context Precision
- **Error Tree:** Output sai -> Context dung? -> Mot phan dung, nhung dung tai lieu cu.
- **Root cause:** BM25 match tot tu "mat khau" nhung khong phan biet policy moi/cu.
- **Suggested fix:** Bo sung metadata `source_priority`, rerank uu tien `v2`, va neu trong context co nhieu version thi prompt phai chon "hien hanh".

### #3
- **Question:** Bao lau phai doi mat khau mot lan?
- **Expected:** Chinh sach hien hanh la 120 ngay; 90 ngay la ban cu.
- **Got:** Rat de bi tra loi 90 ngay vi con so nay thuong duoc lexical search bat manh.
- **Worst metric:** Faithfulness
- **Error Tree:** Output sai -> Context dung? -> Co the co ca 2 context, nhung LLM/fallback lay nham con so cu.
- **Root cause:** Numeric conflict giua hai policy version. Khi khong co LLM, fallback tra ve chunk dau tien; khi co LLM, prompt hien tai chua ep uu tien policy moi.
- **Suggested fix:** Top-k context nen kem ngay hieu luc; them instruction "neu co nhieu version, chi tra loi theo tai lieu moi nhat".

### #4
- **Question:** Mot nhan vien Senior co 9 nam tham nien duoc nghi bao nhieu ngay phep nam va luong trong khoang nao?
- **Expected:** 18 ngay phep va khoang luong Senior P3-P4 la 20-35 trieu VND/thang.
- **Got:** Co nguy co chi tra loi duoc 1 nua cau hoi, hoac lay sai cong thuc tham nien cu.
- **Worst metric:** Context Recall
- **Error Tree:** Output sai/thieu -> Context dung? -> Chua du, vi can ghep nhieu tai lieu.
- **Root cause:** Multi-hop query can noi thong tin tu chinh sach nghi phep va bang luong. Hybrid search hien tai chua co query decomposition.
- **Suggested fix:** Them query rewrite/decomposition, mo rong top-k truoc rerank, va prompt hop nhat context tu nhieu nguon.

### #5
- **Question:** Nghi phep khong luong 20 ngay can ai phe duyet?
- **Expected:** CEO phe duyet; dong thoi nhan vien phai tu dong bao hiem neu nghi khong luong tren 14 ngay.
- **Got:** De chi lay duoc mot phan "CEO phe duyet" ma bo sot rang buoc bao hiem.
- **Worst metric:** Answer Relevancy
- **Error Tree:** Output chua du -> Context dung? -> Co the dung mot phan, nhung answer thieu chi tiet bo sung.
- **Root cause:** Prompt answer hien tai rat ngan, khong ep tong hop day du cac dieu kien kem theo.
- **Suggested fix:** Doi prompt thanh "tra loi day du + note cac dieu kien lien quan", hoac template hoa answer cho policy co threshold.

## Case Study (cho presentation)

**Question chon phan tich:** Nhan vien duoc nghi bao nhieu ngay phep nam?

**Error Tree walkthrough:**
1. Output dung? -> Chua chac, vi de tra loi 12 ngay thay vi 15 ngay.
2. Context dung? -> Mot phan dung, nhung co nguy co lay nham tai lieu v2023.
3. Query rewrite OK? -> Chua co rewrite theo "hien hanh / current policy".
4. Fix o buoc: Retrieval + rerank + metadata versioning.

**Neu co them 1 gio, se optimize:**
- Cai `ragas` va bo sung `OPENAI_API_KEY` de lay metric that.
- Bat metadata version/current cho cac policy co tai lieu cu-moi.
- Khoi dong Qdrant local de bo fallback in-memory, test lai dense retrieval.
- Them query rewrite cho nhom cau hoi version conflict va multi-hop.
