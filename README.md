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


# 세번째 재구성 및 수정 요청사항

## 1. 목적

- step8 페이지의 상단 UI, 버튼, 제목, 페이지네이션, 충족조건·오류·표시 등 시각적, 기능적 배치와 동작을 첨부 그림과 상세 요구에 따라 완전히 수정하는 것.

---

## 2. 상세 요구 및 기능/레이아웃 개선

① **상단 파일다운로드 버튼**
   - 좌측 상단에 “파일 다운로드” 버튼 배치.
   - 버튼 텍스트가 두 줄로 줄바꿈되거나 잘리지 않게 반드시 한 줄로만 표시. (버튼 너비/텍스트 크기 조절)
   - 버튼 주변 여백은 과도하지 않게 적절히 조정.

② **상단 인쇄하기 버튼**
   - 우측 상단에 “인쇄하기” 버튼 배치.
   - 버튼 텍스트가 반드시 한 줄로만 표시되도록, 버튼 너비와 텍스트 크기 조절.
   - 버튼 주변 여백은 과도하지 않게 조정.
   - 인쇄하기 버튼 클릭 시 실제 사용자 환경에서 프린트 창이 열리도록 기능 정상화(window.print() 등 브라우저 표준 방식 완벽 연동, Streamlit 환경에서도 동작 보장).

③ **상단 제목·페이지 번호 배치**
   - “파일 다운로드”와 “인쇄하기” 버튼은 첫 번째 줄(최상단, 좌/우측)에만 위치. (중앙은 비워둠)
   - **두 번째 줄에 중앙 정렬로** “「의약품 허가 후 제조방법 변경관리 가이드라인(민원인 안내서)」[붙임] 신청양식 예시” 제목만 단독 한 줄 배치. 버튼 줄과 분리!
   - **세 번째 줄에는** 페이지 번호(예: “1 / 7”)만 중앙정렬로 제목 아래 별도 한 줄 배치.
   - 어떤 경우에도 버튼 줄에 제목·페이지 번호가 오지 않도록 할 것. (줄바꿈/정렬 필수)

④ **오류 태그/불필요 HTML 출력 방지**
   - 각 페이지 상단에 <br><h5>1. 신청인</h5> 등 HTML 태그가 코드 그대로 노출되지 않게.
   - 표/텍스트 등 실제 데이터만 노출되고, HTML 코드는 모두 렌더링된 결과로만 보여야 함.
   - Streamlit에서 `st.markdown(..., unsafe_allow_html=True)` 등 사용 시 HTML 코드를 그대로 노출하는 버그 해결.

⑤ **충족조건 표 누락 오류 해결**
   - “충족조건” 표는 동일 title_key의 step6 {requirements}를 기반으로, 조건 개수만큼 줄 생성.
   - 각 줄에는 “충족조건” 열과 “조건 충족 여부” 열 포함.
   - 각 조건에 대해 step6_items[title_key]["requirements"][키]의 텍스트, step6_selections[title_key+"_req_"+키]가 “충족”이면 ○, “미충족”이면 ×로 출력.
   - 충족조건 표 내용이 누락 없이 전부 생성되어야 하며, 표의 구조(머릿줄, 각 줄, 정렬)는 첨부 워드 양식과 동일하게 구성.

⑥ **페이지 구조 요약**
   - 1줄: 좌 “파일 다운로드” 버튼 / 우 “인쇄하기” 버튼 (한 줄, 중간 비움)
   - 2줄: 제목 “「의약품 허가 후 제조방법 변경관리 가이드라인(민원인 안내서)」[붙임] 신청양식 예시” (중앙, 단독 한 줄)
   - 3줄: 페이지 번호(“현재/총페이지”) (중앙, 단독 한 줄)
   - 그 이후: 각 페이지별 표/항목 내용(불필요 HTML, 태그, 오류 문구 등 없음, 모든 표는 첨부 워드 양식 구조와 일치)
   - 버튼/제목/페이지번호가 한 줄에 섞여서 나오지 않게 반드시 분리.

⑦ **레이아웃 아스키 예시**

┌──────────────────────────────────────────────┐
│ [파일 다운로드]                     [인쇄하기] │ ← (첫 번째 줄, 양쪽에 버튼. 제목 없음)
│         「의약품 허가 후 제조방법 변경관리 가이드라인(민원인 안내서)」[붙임] 신청양식 예시         │ ← (두 번째 줄, 중앙, 제목만)
│                        1 / 7                 │ ← (세 번째 줄, 중앙, 페이지 번호만)
└──────────────────────────────────────────────┘

---

## 3. 추가 안내

- 각 항목(버튼, 제목, 페이지번호, 인쇄, 표, 오류 등)은 동시에 적용되어야 함.
- 요청한 모든 사항이 100% 일치하게 구현되지 않을 경우 결과물 불인정.
- 각 요구사항의 충족여부를 세부적으로 검증할 예정이므로, 한 줄, 한 칸, 한 글자도 요구와 다르면 안 됨.
- UI 및 기능 구현 시 첨부된 그림(스크린샷)과 완전히 동일한 구조로 배치할 것. (단, 위의 조정안이 더 우선)


# Third Restructuring and Revision Request

## 1. Purpose

- To completely revise the step8 page UI, buttons, title, pagination, satisfaction requirements, error displays, and all visual/functional layouts and behaviors in accordance with the attached screenshots and the detailed requirements below.

---

## 2. Detailed Requirements and UI/Functional Improvements

① **File Download Button (Top-Left)**
   - Place a "[파일 다운로드]" button at the top left.
   - The button label must always appear on a single line (never split/wrapped). Adjust button width or font size as needed.
   - Ensure button margins are not excessive.

② **Print Button (Top-Right)**
   - Place a "[인쇄하기]" button at the top right, on the same horizontal line as the file download button.
   - The label must always display on a single line. Adjust button width or font size if necessary.
   - Margins around the button must not be excessive.
   - The print button must open the actual print dialog on the user's computer (implement using standard browser-compatible approaches such as `window.print()`; must function correctly in Streamlit regardless of environment).

③ **Page Title and Pagination Display**
   - **First Row:** Only the "[파일 다운로드]" (left) and "[인쇄하기]" (right) buttons, with the rest of the row empty.
   - **Second Row:** Centered, display only the page title:
     - "「의약품 허가 후 제조방법 변경관리 가이드라인(민원인 안내서)」[붙임] 신청양식 예시"
     - The title must always be on its own line, not sharing the row with any button or page number.
   - **Third Row:** Directly below the title, center-aligned, display the page number (e.g., "1 / 7") on its own line.
   - Under no circumstances should the buttons, title, or page number be displayed on the same line together. Each element must occupy its own distinct row.

④ **Prevent Raw HTML/Tag/Error Output**
   - Do NOT display any raw HTML tags or unrendered code at the top of each page (e.g., `<br><h5>1. 신청인</h5>...` etc).
   - Only properly rendered tables and text data should be visible to the user.
   - When using Streamlit (e.g., `st.markdown(..., unsafe_allow_html=True)`), ensure that raw HTML is rendered, not printed as literal text.

⑤ **Satisfaction Requirements Table – Omission/Error Fix**
   - The "Satisfaction Requirements" table must:
     - For the current `title_key`, use all requirements from step6 (`step6_items[title_key]["requirements"]`).
     - Generate a row for each requirement, with two columns: "Requirement" and "Met or Not Met".
     - For each row:
       - Display the requirement text.
       - Display "○" if `step6_selections[title_key+"_req_"+key] == "충족"`, or "×" if "미충족".
   - No requirement row may be omitted.
   - Table structure (headers, rows, alignment) must strictly match the official Word form template.

⑥ **UI Layout Summary**
   - Row 1: Left – [파일 다운로드] button, Right – [인쇄하기] button. No title or page number here.
   - Row 2: Centered page title only (Korean as above), as a single line.
   - Row 3: Centered page number (e.g., "1 / 7"), as a single line.
   - Following rows: Page content, tables, etc. (No raw HTML, tags, or extraneous error messages.)
   - Buttons, title, and page number must never share a row – always separate.

⑦ **UI Layout Example (ASCII Art)**

┌──────────────────────────────────────────────┐
│ [파일 다운로드]                     [인쇄하기] │ ← (First row: both buttons on sides, no title)
│         「의약품 허가 후 제조방법 변경관리 가이드라인(민원인 안내서)」[붙임] 신청양식 예시         │ ← (Second row: centered title only)
│                        1 / 7                 │ ← (Third row: centered page number only)
└──────────────────────────────────────────────┘

---

## 3. Additional Notes

- All changes (buttons, title, page number, print, table rendering, error prevention) must be applied together.
- Output will be rejected if any single requirement here is not followed exactly.
- All details (line, cell, margin, header, table, etc.) will be strictly checked – any deviation will result in rejection.
- The UI and layout must match the attached screenshots precisely unless a more explicit textual instruction above overrides them.





