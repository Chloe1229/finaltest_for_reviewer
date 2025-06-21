# Manufacturing Change Guidance Tool

This repository contains a Streamlit application that assists with Korean pharmaceutical manufacturing change submissions. The main script is `step1_to_8_step8_final_.py`, which guides users through a multi-step questionnaire and generates a Word form based on the results.

## Disclaimer
- This tool is a personal research project built from publicly available resources. It does **not** represent an official regulatory position.
- All content is based on the guidelines valid as of **June 21, 2025**. Regulations may change, so always verify the latest requirements.

## Installation
1. Create a Python environment.
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Running
Start the Streamlit app from the repository root:
```bash
streamlit run step1_to_8_step8_final_.py
```

## Workflow Overview
The application consists of the following steps. Navigation buttons are provided on each page.

1. **Cover Page** – Introductory notes and a start button.
2. **Step 1** – Confirm whether the product is subject to CTD submission.
3. **Step 2** – Check if the change concerns manufacturing items in CTD sections.
4. **Step 3** – Verify that a CTD-formatted dossier has been submitted previously.
5. **Step 4** – Select CTD sections (e.g., `3.2.S`, `3.2.P`, Design Space) relevant to the change.
6. **Step 5** – Choose specific change categories for each selected section.
7. **Step 6** – Indicate whether each requirement is satisfied.
8. **Step 7** – View the resulting classification and required documents.
9. **Step 8** – Download a generated Word application form and view the same table on screen. Pages are provided per `(title_key, result_index)` pair, and a message is shown when no result exists for a page.
10. **Step 9** – Download the guideline PDF and Q&A document.

The Word template `제조방법변경 신청양식_empty_.docx` is used to ensure the generated document matches the official format. The guideline PDFs are included for reference.

## Output Files
- **PDFs** – Located in the repository: `의약품 허가 후 제조방법 변경관리 가이드라인(민원인안내서).pdf` and `의약품 허가 후 제조방법 변경관리 질의응답집(민원인안내서).pdf`.
- **Word form** – Generated dynamically in Step 8 and offered for download.

## License
All automation logic, UI, and data structure in this project are © 2025 Chloe Kim. Redistribution or reuse is prohibited.
