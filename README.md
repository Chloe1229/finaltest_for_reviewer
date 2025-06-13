# 첫번째 재구성 및 수정 요청사항

## 1. 목적 및 배경
- step8 페이지에서의 페이징 및 표 생성은 반드시 첨부된 워드 신청서 양식(공식 문서) 구조를 따라야 함.
- 기존에는 title_key 단위로 한 페이지만 생성되었으나,
- 실제 Step7 하드코딩 결과(output_1_text, output_2_text)가 동일한 title_key에서 복수(2개 이상) 생성될 수 있음.
- 즉, 같은 title_key에서 output_1_text/output_2_text 조합이 다르면, 해당 조합마다 별도의 페이지와 표를 생성해야 함.

---

## 2. 데이터 구조 요구

### [A] step7_results 구조 표준화
- 기존:
    step7_results = {
        "s2_2": {
            "title_text": "...",
            "output_1_tag": "...",
            "output_1_text": "...",
            "output_2_text": "..."
        },
        # ... (title_key별 단일 결과)
    }
- 변경:
    - title_key별로 복수 결과가 리스트로 존재해야 함
    - 예시:
        step7_results = {
            "s2_2": [
                {
                    "title_text": "...",
                    "output_1_tag": "...",
                    "output_1_text": "...",
                    "output_2_text": "..."
                },
                {
                    "title_text": "...",
                    "output_1_tag": "...",
                    "output_1_text": "...",
                    "output_2_text": "..."
                }
            ],
            # ...
        }
- 반드시 모든 title_key 값이 리스트(list)로 존재해야 하며, 결과가 1개여도 리스트로 래핑함.

---

## 3. 페이징/페이지 순회 구조

- step8의 페이지 리스트는 title_key, result_index 쌍의 일련번호로 구성해야 함.
- 예시:
    page_list = []
    for title_key, results in step7_results.items():
        for idx in range(len(results)):
            page_list.append((title_key, idx))
    # page_list 예시: [("s2_2", 0), ("s2_2", 1), ("p3_13", 0), ...]

---

## 4. 화면 및 워드 파일 표 생성 로직

- 각 페이지는 page_list[현재 인덱스]의 (title_key, idx) 쌍을 참조하여
    - step7_results[title_key][idx]에 있는 output_1_text, output_2_text 값으로 표를 생성
    - 표 구조 및 제목, 항목 등은 첨부된 워드양식과 일치해야 하며,
    - 나머지 신청인, 충족조건, 필요서류 등 모든 데이터 매핑은 기존 로직과 동일하게 적용

---

## 5. Streamlit 및 워드 파일 동기화

- 화면에서 표시되는 표와 다운로드되는 워드 파일은 반드시 동일한 페이지/결과를 사용
- 다운로드 버튼 및 인쇄 버튼 등은 기존대로 페이지 단위로 연동

---

## 6. 예외처리 및 기타

- 기존 title_key별로 결과가 0개일 경우는 페이지 생성에서 제외
- 복수 결과가 있을 때만 여러 페이지로 분기, 단일 결과도 항상 리스트 기반으로 접근
- 모든 반복문, 인덱싱, 표 생성 함수에서 title_key, idx 쌍을 기준으로 호출

---

## 7. 필수 코드 예시

### (1) step7_results 구조 처리

for k, v in step7_results.items():
    if not isinstance(v, list):
        step7_results[k] = [v]  # 결과 1개도 리스트화

page_list = []
for title_key, results in step7_results.items():
    for idx in range(len(results)):
        page_list.append((title_key, idx))

### (2) 현재 페이지별 데이터 접근

current_title_key, current_result_idx = page_list[current_page]
result = step7_results[current_title_key][current_result_idx]
# result["output_1_text"], result["output_2_text"] 등 사용

### (3) 표 생성 함수 (워드/화면 공통)

def render_result_table(title_key, result):
    # result는 step7_results[title_key][idx]의 dict
    # 표 생성 로직 (신청인, 변경유형, 신청유형, 충족조건, 필요서류 등)
    ...

---

## 8. 수정 대상/위치

- step8에서 페이지 처리하는 모든 부분(페이지 리스트, 네비게이션, 데이터 바인딩, 표 생성 등)
- 워드 파일 생성 함수(create_application_docx 등)도 동일 로직 반영

---

## 9. 결과 확인

- 동일 title_key에서 output_1_text/output_2_text가 여러 개 나오면, 각 조합별로 완전히 별도 페이지/표/워드파일이 생성되는지 반드시 검증

---

## 10. 요약

- title_key별 1페이지 → title_key+output조합별 1페이지로 구조를 변경
- 모든 반복, 인덱싱, 표, 워드생성은 (title_key, idx) 기반으로만 작동
- 데이터 접근, 표생성, 다운로드 등 모든 부분에서 이 구조를 일관 적용
- 코드 내 모든 참조부/반복부는 반드시 이 구조 반영

---

상기 사항 미준수 시 결과 인정 불가.
(불분명할 경우 본 작업지시서 예시/구조/용어만 엄격 준수)



# 첫번째 재구성 및 수정 요청사항(Eng. ver) First Restructuring and Revision Request

## 1. Purpose and Background
- The paging and table generation in the step8 page must strictly follow the structure of the attached official Word application form template.
- Previously, only one page was generated per title_key.
- However, in actual Step7 hardcoded results (output_1_text, output_2_text), **multiple results (two or more) can be generated for the same title_key**.
- **In other words, even if the title_key is the same, if the output_1_text/output_2_text combination differs, a separate page and table must be generated for each combination.**

---

## 2. Data Structure Requirements

### [A] Standardization of step7_results Structure
- **Previous structure:**  
    ```python
    step7_results = {
        "s2_2": {
            "title_text": "...",
            "output_1_tag": "...",
            "output_1_text": "...",
            "output_2_text": "..."
        },
        # ... (single result per title_key)
    }
    ```
- **Revised structure:**  
    - Each title_key must have a **list of results** (even if there is only one result)
    - Example:
    ```python
    step7_results = {
        "s2_2": [
            {
                "title_text": "...",
                "output_1_tag": "...",
                "output_1_text": "...",
                "output_2_text": "..."
            },
            {
                "title_text": "...",
                "output_1_tag": "...",
                "output_1_text": "...",
                "output_2_text": "..."
            }
        ],
        # ...
    }
    ```
- **All title_key values must always be lists**. If there is only one result, it must still be wrapped in a list.

---

## 3. Paging / Page Iteration Structure

- The page list in step8 must be constructed as a **tuple of (title_key, result_index)** for each result.
- Example:
    ```python
    page_list = []
    for title_key, results in step7_results.items():
        for idx in range(len(results)):
            page_list.append((title_key, idx))
    # Example: [("s2_2", 0), ("s2_2", 1), ("p3_13", 0), ...]
    ```

---

## 4. Screen and Word File Table Generation Logic

- For each page, reference the (title_key, idx) pair from page_list[current_page_index]
    - Use step7_results[title_key][idx] for output_1_text and output_2_text for table generation.
    - The table structure, headings, and fields must match the attached Word template as closely as possible.
    - Other fields such as applicant info, satisfaction conditions, and required documents are mapped as per existing logic.

---

## 5. Streamlit and Word File Synchronization

- The table displayed on screen and the Word file downloaded **must use exactly the same page/result data**.
- The download and print buttons must remain page-specific, exactly as currently implemented.

---

## 6. Exception Handling and Additional Details

- If a title_key has zero results, that title_key must be excluded from page generation.
- If there are multiple results for a title_key, generate a page for each result. If there is only one, still use the list-based approach.
- All loops, indexing, and table generation functions must use the (title_key, idx) tuple for data referencing.

---

## 7. Required Code Examples

### (1) Ensuring step7_results Structure

```python
# Make sure all title_key values are lists
for k, v in step7_results.items():
    if not isinstance(v, list):
        step7_results[k] = [v]  # Wrap single result in a list

# Build page_list
page_list = []
for title_key, results in step7_results.items():
    for idx in range(len(results)):
        page_list.append((title_key, idx))

### (2) Accessing Data for the Current Page
current_title_key, current_result_idx = page_list[current_page]
result = step7_results[current_title_key][current_result_idx]
# Use result["output_1_text"], result["output_2_text"], etc.


### (3) Table Generation Function (for both Word and Screen)
def render_result_table(title_key, result):
    # result is step7_results[title_key][idx]
    # Build the table: Applicant, Change Type, Application Type, Satisfaction Conditions, Required Documents, etc.
    ...


## 8. Targets/Locations for Modification
All sections of step8 that handle page iteration (page list, navigation, data binding, table rendering, etc.)
The Word file generation function (such as create_application_docx) must also be updated to use the same logic.

## 9. Result Verification
If multiple output_1_text/output_2_text combinations are generated for the same title_key, there must be a fully separate page, table, and Word file for each combination.
Test with multi-result and single-result cases to ensure all logic is robust.

## 10. Summary
Change from "one page per title_key" to "one page per output combination (title_key, idx)".
All loops, indexing, table generation, and Word creation must be based exclusively on (title_key, idx).
Data access, table rendering, download functions, etc. must all use this structure consistently.
All references, iterations, and function calls in the code must strictly follow this new pattern.



# 두번째 재구성 및 수정 요청사항

## 1. 목적

- 첫번째 재구성 및 수정 요청사항에서 요구한 기준에 따라, step7에서 해당 (title_key, 조건) 조합에 맞는 도출 결과가 없는 경우 step8의 동작을 엄격히 규정하기 위함.

---

## 2. 요구 동작

- 특정 (title_key, 조건)에 대해 step7에서 도출 결과가 없을 경우(output_1_text와 output_2_text가 모두 없는 경우):
    - 해당 step8 페이지에서는 워드파일 다운로드, 인쇄하기 기능을 **제공하지 않는다**.
    - 이 페이지에서 **화면에 표시되는 내용은 아래 문구 하나만**(줄바꿈 포함) 노출되어야 한다.

        해당 변경사항에 대한 충족조건을 고려하였을 때,
        「의약품 허가 후 제조방법 변경관리 가이드라인」에서 제시하고 있는
        범위에 해당하지 않는 것으로 확인됩니다.

---

## 3. 화면 출력 로직

- step8의 어느 페이지에서든 step7에서 현재 (title_key, 조건)에 맞는 결과가 없으면:
    - **절대 표시하지 말아야 할 것:**
        - 워드파일 다운로드 버튼
        - 인쇄 버튼
        - 어떠한 표/양식 필드도
    - **오직 위 지정 문구만** 출력해야 함(위치·줄바꿈 등 포함).

---

## 4. 페이징/이동

- 이 “결과 없음” 페이지도 step8 페이징에 포함되어야 하며, 다른 페이지처럼 “이전/다음” 네비게이션은 정상적으로 동작해야 한다.

---

## 5. 코드 예시

# step8 출력 로직 예시(Python 유사 코드):
if not has_matching_step7_result:
    st.write(
        "해당 변경사항에 대한 충족조건을 고려하였을 때,\n"
        "「의약품 허가 후 제조방법 변경관리 가이드라인」에서 제시하고 있는\n"
        "범위에 해당하지 않는 것으로 확인됩니다"
    )
    # 다운로드/인쇄/표는 절대 출력 금지
else:
    # 정상 표/버튼/출력 로직
    ...

---

## 6. 요약

- step7에 해당 (title_key, 조건) 조합 결과가 없는 경우
    - step8 화면에는 반드시 지정 문구만 노출
    - 워드파일 다운로드 및 인쇄 기능은 비활성화/숨김 처리
    - 표/양식 그 어떤 것도 표시하지 않음
- 이 로직은 “결과 없음” 페이지에 대해 예외 없이 엄격히 적용되어야 한다.

---

상기 요구사항 미준수시 결과물은 인정되지 않음.  
예시/지침/메시지내용은 반드시 완전히 일치하게 적용할 것.

---



# 두번째 재구성 및 수정 요청사항(Eng. ver) Second Restructuring and Revision Request

## 1. Purpose

- To define strict behavior for step8 when no matching output result exists for a given (title_key, condition) combination in step7, according to the requirements of the first restructuring request.

---

## 2. Required Behavior

- When, for a given (title_key, condition), no output result is generated in step7 (i.e., no matching output_1_text and output_2_text is present):
    - On the relevant step8 page, do not provide Word file download or print functions.
    - The only content to be displayed on the screen for this page must be the following message (with line breaks preserved):

        Considering the satisfaction conditions for this change,
        it has been confirmed that it does not fall within the scope
        presented in the "Guidelines for Post-Approval Changes in Drug Manufacturing Methods."

    - (Korean original for reference:  
        "해당 변경사항에 대한 충족조건을 고려하였을 때,\n「의약품 허가 후 제조방법 변경관리 가이드라인」에서 제시하고 있는\n범위에 해당하지 않는 것으로 확인됩니다")

---

## 3. Screen Display Logic

- For any step8 page where no result from step7 matches the current (title_key, condition):
    - Do NOT display:
        - Word file download button
        - Print button
        - Any table or form fields
    - DO display only the specified message (see above).

---

## 4. Paging/Navigation

- This "no result" page must still be included in the step8 paging sequence, just like any other page.
- Navigation ("Previous", "Next") must work as usual.

---

## 5. Code Example

# Pseudocode for step8 display logic:
if not has_matching_step7_result:
    st.write(
        "Considering the satisfaction conditions for this change,\n"
        "it has been confirmed that it does not fall within the scope\n"
        "presented in the \"Guidelines for Post-Approval Changes in Drug Manufacturing Methods.\""
    )
    # Do NOT render download/print buttons or tables
else:
    # Normal rendering of table, download, print, etc.
    ...

---

## 6. Summary

- When there is no matching result in step7 for the current (title_key, condition),
    - step8 screen must only display the designated message,
    - Word file download and print options are disabled/hidden for that page,
    - no table or form is rendered.
- This logic must be strictly and consistently applied for all cases where no step7 result is found for the page.

---

Failure to fully comply with these requirements will result in rejection of the output.
Follow the instructions, examples, and message content exactly as provided.

---


