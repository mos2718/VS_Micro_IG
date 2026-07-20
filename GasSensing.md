# FHIR Gas Sensing 規格 (Micro-IG)

本文件定義環境感測數據中「空氣品質與氣體感測 (Air Quality and Gas Sensing)」之 FHIR Observation 紀錄規格。  
適用於 PM2.5、TVOC、CO2、CO 等單一數值 (single value) 之感測數據交換與整合。

---

## FHIR Observation JSON Template

下列 JSON 模板，可用於紀錄 PM2.5、TVOC、CO2、CO 等單一數值之氣體感測數據：

```json
{
  "resourceType": "Observation",
  "status": "final",
  "category": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/observation-category",
          "code": "exam",
          "display": "exams"
        }
      ]
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "{{diff.code}}",
        "display": "{{diff.code_display}}"
      }
    ]
  },
  "subject": {
    "reference": "Location/{{location_id}}"
  },
  "effectiveDateTime": "{{input.measure_time}}",
  "valueQuantity": {
    "value": {{input.value}},
    "unit": "{{diff.value_unit}}",
    "system": "http://unitsofmeasure.org",
    "code": "{{diff.value_code}}"
  }
}
```


此 JSON 模板當中，有些數值固定，如：
- `status` 固定為 `"final"`
- `category.code` 固定為 `"vital-signs"`

有些數值標記為變數，格式如 `{{diff.code}}` 及 `{{input.patient_id}}`。  
其中 `diff` 起頭之變數，代表需套入 PM2.5、TVOC、CO2、CO 等量測項目之專屬代碼與單位。

---

## diff.csv

定義不同氣體感測項目對應之標準 LOINC 代碼與 UCUM 單位：

```csv
id,code,code_display,value_unit,value_code
pm25,80389-1,PM2.5 [Mass/volume] in Air,ug/m3,ug/m3
tvoc,83325-2,Volatile organic compounds [Mass/volume] in Air,ug/m3,ug/m3
co2,19936-4,Carbon dioxide [Volume Fraction] in Air,ppm,ppm
co,20562-5,Carbon monoxide [Volume Fraction] in Air,ppm,ppm
```

**代碼說明：**
* **PM2.5**: LOINC `80389-1`，單位為微克每立方米 (ug/m3)
* **TVOC**: LOINC `83325-2`，單位為微克每立方米 (ug/m3)
* **CO2**: LOINC `19936-4`，單位為百萬分比濃度 (ppm)
* **CO**: LOINC `20562-5`，單位為百萬分比濃度 (ppm)

---

## JSON 模板中 input 要求

JSON 模板中 `input` 起頭之變數，應由程式帶入或從感測器介面輸入，如下：

- `{{{location_id}}`：環境/空間位置
- `{{input.measure_time}}`：感測數據紀錄時間或設備採樣時間 (ISO 8601 格式，例：`2026-07-20T14:30:00+08:00`)
- `{{input.value}}`：感測器實際量測之數值 (Number 類型，無需加引號)

---

### 💡 規格設計說明：
1. **標準化代碼**：採用國際醫療與健康資訊標準 **LOINC** 作為感測項目代碼，並使用 **UCUM** (Unified Code for Units of Measure) 作為單位代碼，確保跨系統互通性。
2. **彈性主體 (Subject)**：雖然模板保留 `Patient`，但在註解與範例中特別說明，環境感測更適合使用 `Location` 或 `Device` 作為 Reference，符合 FHIR 對環境監測的建模最佳實踐。
3. **擴充性**：額外提供了 `component` 陣列的進階用法，方便未來若有「多合一空氣品質感測器」時，能直接套用打包上傳的架構。