  
CREATE procedure [dbo].[Usp_TariffUploadDoc_FillDetails](@ProviderID int,@MOUID varchar(max) ,@TariffDocId Int,@Type varchar(10)='Mapped' )      
as        
begin        
set nocount ON     
   
  
 If @TariffDocId > 0   
 Begin  
  Select mp.ProviderId  ,FileName,SystemFileName,case when OMp.id is not null then 'yes' else 'no'end IsOldDoc   
  from ProviderTariffDocs (nolock) Doc Inner Join   
  ProviderTariff_Map (nolock) Mp On Doc.Id=Mp.DocumentId  
  Left join (select * from ProviderTariff_Map (nolock) where mouid= 0) OMp On doc.id=omp.documentid   
  Where Mp.Id=@TariffDocId  
 End  
   
Else if @ProviderID > 0 and @MOUID != ''and @MOUID != '-1' and @Type = 'NotMapped'  
 Begin  
 Select Mp.Id FileId,Mp.MOUID,Mp.Documentid  
   , Row_Number() over(order by Mp.MOUID Desc,TariffDoc.ID Desc) SlNo  
   ,FileName, MU.Name as UserName,TariffDoc.CreatedDateTime UpdateDate  
   ,SystemFileName into #temp  
   from  ProviderTariffDocs (nolock) TariffDoc   
   Join ProviderTariff_Map Mp On TariffDoc.Id=Mp.DocumentID  
   JOIN Lnk_UserRegions (NOLOCK) LU ON LU.ID = Mp.CREATEDUSERREGIONID  
   JOIN Mst_Users (NOLOCK) MU ON MU.ID= LU.USERID  
   where  Mp.ProviderID=@ProviderID and Mp.Status=1   
   and TariffDoc.Status=1   
   and MOUID IN (SELECT items FROM dbo.Split(@MOUID, ','))  
  
    
 Select max(Mp.Id) as FileId,0 MOUID,Mp.Documentid  
   , Row_Number() over(order by Mp.Documentid Desc) SlNo  
   ,FileName,TariffDoc.CreatedDateTime UpdateDate  
   ,SystemFileName   
   from  ProviderTariffDocs (nolock) TariffDoc   
   Join ProviderTariff_Map Mp On TariffDoc.Id=Mp.DocumentID  
   JOIN Lnk_UserRegions (NOLOCK) LU ON LU.ID = Mp.CREATEDUSERREGIONID  
   JOIN Mst_Users (NOLOCK) MU ON MU.ID= LU.USERID  
   where  Mp.ProviderID=@ProviderID --and Mp.Status=1   
   and TariffDoc.Status=1   
   and MOUID NOT IN (SELECT items FROM dbo.Split(@MOUID, ','))  
   and Mp.Documentid not in (select documentid from #temp)  
   Group by Mp.Documentid,FileName,TariffDoc.CreatedDateTime,systemfilename  
  order by  UpdateDate  desc  
  drop table #temp  
 End  
  
Else if @ProviderID > 0 and @MOUID != ''and @MOUID != '-1'  
 Begin  
   Select Mp.Id FileId,Mp.MOUID  
   , Row_Number() over(order by Mp.MOUID Desc,TariffDoc.ID Desc) SlNo  
   ,FileName, MU.Name as UserName,TariffDoc.CreatedDateTime UpdateDate  
   ,SystemFileName   
   from  ProviderTariffDocs (nolock) TariffDoc   
   Join ProviderTariff_Map Mp On TariffDoc.Id=Mp.DocumentID  
   JOIN Lnk_UserRegions (NOLOCK) LU ON LU.ID = Mp.CREATEDUSERREGIONID  
   JOIN Mst_Users (NOLOCK) MU ON MU.ID= LU.USERID  
   where  Mp.ProviderID=@ProviderID and Mp.Status=1   
   and TariffDoc.Status=1   
   and MOUID IN (SELECT items FROM dbo.Split(@MOUID, ','))  
  order by  UpdateDate  desc  
 End  
Else  
 Begin  
  --Select Row_Number() over(order by FileName asc,Id Asc) SlNo ,  
  --Id FileId,FileName,SystemFileName,CreatedDateTime UpdateDate from ProviderTariffDocs (nolock)PDocs  
  --Where Status=1  order by  CreatedDateTime   desc  
  
  --SELECT Row_Number() over(order by FileName asc,PDOCS.ID  Asc) SlNo ,  
  --MAX(PDOCS.ID) FileId,FileName,SystemFileName,PDOCS.CREATEDDATETIME UpdateDate,MAX(MP.ID) TariffDocId  
  --FROM PROVIDERTARIFFDOCS (NOLOCK) PDOCS  
  --INNER JOIN PROVIDERTARIFF_MAP MP ON PDOCS.ID = MP.DOCUMENTID  
  --WHERE PDOCS.STATUS=1 AND MP.STATUS = 1 and ProviderID=@ProviderID   
  --GROUP BY FILENAME,SYSTEMFILENAME,PDOCS.CREATEDDATETIME ,PDOCS.id  
  --order by  UpdateDate desc  
  SELECT Row_Number() over(order by FileName asc,PDOCS.ID  Asc) SlNo ,  
  MAX(PDOCS.ID) FileId,FileName,SystemFileName,PDOCS.CREATEDDATETIME UpdateDate,MAX(MP.ID) TariffDocId,MU.NAME UserName  
  FROM PROVIDERTARIFFDOCS (NOLOCK) PDOCS  
  JOIN PROVIDERTARIFF_MAP (NOLOCK) MP ON PDOCS.ID = MP.DOCUMENTID  
  JOIN Lnk_UserRegions (NOLOCK) LU ON LU.ID = PDOCS.CREATEDUSERREGIONID  
  JOIN Mst_Users (NOLOCK) MU ON MU.ID= LU.USERID  
    
  WHERE PDOCS.STATUS=1 and ProviderID=@ProviderID   
  GROUP BY FILENAME,SYSTEMFILENAME,PDOCS.CREATEDDATETIME ,PDOCS.id,MU.NAME  
  order by  UpdateDate desc  
  
  
 End  
       
End  
  
  
  
  
  
  







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
