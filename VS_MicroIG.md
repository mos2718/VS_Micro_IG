# Vital signs Micro IG 
- 訂立 vital signs 中體溫、心率、血氧、呼吸  Micro-IG
-- 包含 FHIR observation JSON 欄位定義檔，以及體溫、心率、血氧、呼吸內容差異 CSV diff.csv

## FHIR observation JSON Template
下列 JSON 模板，可用於紀錄體溫、心律、血氧、呼吸等單一數值(single value) 之 vital signs
```json=
{
  "resourceType": "Observation",
  "status": "status",
  "category": [
    {"coding": [
        { "system": "http://terminology.hl7.org/CodeSystem/observation-category",
          "code": "vital-signs"    }  ]  }
  ],
  "code": {
    "coding": [
      { "system": "http://loinc.org",
        "code": "{{diff.code}}",
        "display":"{{diff.code_display}}" 
      }
    ]
  },
  "subject": {
    "reference": "Patient/{{input.patient_id}}"
  },
  "effectiveDateTime": "{{input.measure_time}}",
  "valueQuantity": {
    "value": "{{input.value}}",
    "unit": "{{diff.value_unit}}",
    "system": "http://unitsofmeasure.org",
    "code":"{{diff.value_code}}" 
  }
}
```
此 JSON 模板當中，有些數值固定，如:
category.code 固定為 "vital-signs"
status  固定為  "final"
有些數值標記為變數，格式如 {{diff.code}}" 及 {{input.patient_id}}。
其中 diff 起頭之變數，表需套入體溫、心律、血氧、呼吸等量測差異之代碼、單位等
另外，避免 data comsumer 處理 option 欄位的困擾(程式需增加許多條件判斷，處理麻煩，容易出錯)，此規格不提供option 欄位

JSON 模板中， diff.csv 定義體溫、心律、血氧、呼吸等紀錄的差異內容:
### diff.csv
```csv=
id,code,code_display,value_unit,value_code
body-temperature,8310-5,Body temperature,degC,Cel
heart-rate,8867-4,Heart rate,/min,/min
respiratory-rate,9279-1,Respiratory rate,/min,/min
spo2,59408-5,Oxygen saturation in Arterial blood by Pulse oximetry,%,%
```

## JSON 模板中 input 要求
JSON 模板中 input 起頭之變數，應由程式帶入或輸入，如下:
- {{input.patient_id}} 病人 id 
- {{input.measure_time}} 可抓取紀錄時間或電腦時間
- {{input.value}}  可轉換儀器產生的資料或由介面輸入
FHIR server URL : https://hapi.fhir.org/baseR4/  。 請設計生理監測輸入網頁。輸入後，點選網頁 button， 產生心跳之生理監測資料，並上傳 FHIR server

