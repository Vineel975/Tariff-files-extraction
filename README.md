Old doc : 2_Suraksha_Hospital_01_04_2016_30_04_2030_133879728436966317.xls

New doc : 297_Apollo_Hospitals__Jubilee_Hills__29_12_2019_31_12_2030_134193248271137546.pdf

  
SELECT TOP 20
    doc.Id,
    doc.FileName,
    doc.SystemFileName,
    doc.CreatedDateTime,
    mp.ProviderID,
    mp.MOUID,
    CASE WHEN omp.id IS NOT NULL THEN 'yes' ELSE 'no' END AS isOldDoc
FROM ProviderTariffDocs doc WITH (NOLOCK)
JOIN ProviderTariff_Map mp WITH (NOLOCK) ON doc.Id = mp.DocumentId
LEFT JOIN (
    SELECT * FROM ProviderTariff_Map WITH (NOLOCK) WHERE MOUID = 0
) omp ON doc.Id = omp.DocumentId
WHERE doc.Status = 1
  AND mp.Status = 1
  AND doc.FileName LIKE '%.pdf'
ORDER BY doc.CreatedDateTime DESC


SELECT 
    CASE WHEN omp.id IS NOT NULL THEN 'yes' ELSE 'no' END AS isOldDoc,
    COUNT(*) AS Count,
    MIN(doc.SystemFileName) AS SampleFileName
FROM ProviderTariffDocs doc WITH (NOLOCK)
JOIN ProviderTariff_Map mp WITH (NOLOCK) ON doc.Id = mp.DocumentId
LEFT JOIN (
    SELECT * FROM ProviderTariff_Map WITH (NOLOCK) WHERE MOUID = 0
) omp ON doc.Id = omp.DocumentId
WHERE doc.Status = 1
GROUP BY CASE WHEN omp.id IS NOT NULL THEN 'yes' ELSE 'no' END
```

From the results you'll see `SystemFileName` values. Then go to your S3 bucket and search for one of those filenames — the folder path before the filename is your answer.

For example if `SystemFileName = 'abc123.pdf'` and in S3 you find it at:
```
prod-spectra-app-s3-provider-docs/
  1234/TariffDocument/abc123.pdf     ← isOldDoc = 'no'
```
Then `PROVIDER_TARIFF_DOCUMENT_PATH = TariffDocument/`

And if `isOldDoc = 'yes'` and it's at:
```
  OldWebShare/xyz456.pdf             ← isOldDoc = 'yes'


  SELECT 
    CASE WHEN omp.id IS NOT NULL THEN 'yes' ELSE 'no' END AS isOldDoc,
    COUNT(*) AS Count,
    MIN(doc.SystemFileName) AS SampleFileName
FROM ProviderTariffDocs doc WITH (NOLOCK)
JOIN ProviderTariff_Map mp WITH (NOLOCK) ON doc.Id = mp.DocumentId
LEFT JOIN (
    SELECT * FROM ProviderTariff_Map WITH (NOLOCK) WHERE MOUID = 0
) omp ON doc.Id = omp.DocumentId
WHERE doc.Status = 1
GROUP BY CASE WHEN omp.id IS NOT NULL THEN 'yes' ELSE 'no' END
  
  
  
  
  
  







<add key="PROVIDER_DOC_BUCKET_NAME"               value="prod-spectra-app-s3-provider-docs" />
<add key="PROVIDER_DOC_BUCKET_REGION"              value="ap-south-1" />
<add key="PROVIDER_TARIFF_DOCUMENT_PATH"           value="TariffDocs/" />
<add key="PROVIDER_TARIFF_DOCUMENT_PATH_WEBSHARE"  value="OldTariffDocs/" />
<add key="AWS_ACCESS_KEY_ID"                       value="AKIAXAF7KDNNNDS2773J" />
<add key="AWS_SECRET_ACCESS_KEY"                   value="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" />


SELECT TOP 20
    td.FileId,
    td.ProviderId,
    td.FileName,
    td.SystemFileName,
    td.isOldDoc,
    td.UpdateDate
FROM TariffUploadDoc td WITH (NOLOCK)
WHERE td.FileName LIKE '%.pdf'
  AND ISNULL(td.Deleted, 0) = 0
ORDER BY td.UpdateDate DESC


Required env vars:
1. MEMBER_DB_USER
2. MEMBER_DB_SERVER
3. MEMBER_DB_DATABASE
4. MEMBER_DB_PASSWORD
5. PROVIDER_DOC_BUCKET_NAME
6. PROVIDER_TARIFF_DOCUMENT_PATH
7. PROVIDER_TARIFF_DOCUMENT_PATH_WEBSHARE
Optional env vars:
1. MEMBER_DB_PORT
2. MEMBER_DB_DOMAIN
3. MEMBER_DB_USERNAME
4. PROVIDER_DOC_BUCKET_REGION

PROVIDER_DOC_BUCKET_NAME=prod-spectra-app-s3-provider-docs
PROVIDER_DOC_BUCKET_REGION=ap-south-1
MEMBER_DB_SERVER = spectra-db-qa.fhpl.in
MEMBER_DB_DATABASE = McarePlus_QA
MEMBER_DB_USER = FHPL\satyavineel.k
MEMBER_DB_PASSWORD = Cabbage@2001
