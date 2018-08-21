# AAfun manual
ASReml-R Added functions

##### Attension: AAfun is now available and works for ASReml-R V3.0 or ASReml-R V4.0. The online version AAfun is only for ASReml-R V3.0. The new version AAfun4 works for ASReml-R V4.0, but not supplied online in present. If readers feel interesting in AAfun4, you can get the package AAfun4 by sending email to me (yzhlinscau@163.com).

## About AAfun

The breedRPlus is builded on the base of package `'asreml'` for some additional functions, such as calculating standard error (se), running batch analysis, getting cov/var/corr matrix for FA models, etc. 

## INSTALL package
``` r
install.packages(c('amap',"agricolae","agridat","grid",'ggplot2',"gplots",
"devtools","MCMCglmm",'nadiv',"plyr","reshape2","sqldf"))

devtools::install_github('yzhlinscau/AAfun')
``` 

## AAfun function

  - pin() to calculate heritability or corr with se;
  - asreml.batch() to run batch analysis;
  - model.comp() to run model comparisons;
  - met.corr() to get cov/var/corr matrix for FA models;
  - met.plot() to plot MET data;
  - met.biplot() to run biplot for MET factor analytic results;
  - etc...

## DEMO functions
``` r
library(asreml)
library(AAfun  
demo('pin')
```
## DEMO data
#### data1 with no pedigree
``` r
data(PrSpa)
df<-PrSpa
```
#### data2 with pedigree
``` r
data(dfm2)
df2<-dfm2
``` 
## function 1 pin():calculate se for h2 or corr
##### exmaple 1.1 for sigle trait model
``` r
fm<-asreml(h5~1+Rep, random=~Fam, subset=Spacing=='3',data=df,trace=F)

summary(fm)$varcomp[,1:3]
``` 
calculate heritability:
``` r
pin(fm, h2 ~4*V1/(V1+V2),signif=T) 
``` 
##### exmaple 1.2 for us model for bi-trait
``` r
fm2<-asreml(cbind(dj,h5)~ trait+trait:Rep,
               random=~ us(trait):Fam, rcov=~units:us(trait),
               subset=Spacing=='3',data=df,maxit=40,trace=F)

summary(fm2)$varcomp[,1:3]
``` 
calculate heritability for both traits:
``` r
pin(fm2, h2_A ~ 4 * V1/(V1+V5)) 

pin(fm2, h2_B ~ 4 * V3/(V3+V7)) 
``` 
calculate genetic and phenotypic corr between traits:
``` r
pin(fm2, gCORR ~ V2/sqrt(V1*V3),signif=TRUE) 

pin(fm2, pCORR ~ (V2+V6)/sqrt((V1+V5)*(V3+V7)),signif=TRUE) 
``` 
##### exmaple 1.3 for corr model for multi-trait
``` r
fm3<-asreml(cbind(dj,h3,h5)~ trait+trait:Rep, 
            random=~ corgh(trait):Fam, rcov=~units:us(trait),
            subset=Spacing=='3',data=df,maxit=40,trace=F)
            
summary(fm3)$varcomp[,1:3]
``` 
return corr results:
``` r
pin(fm3,corN=3) 
```
just return the first corr:
``` r
pin(fm3) 
``` 
pin() also works for data with pedigree files.

## function 2 asreml.batch():batch analysis
#### exmaple 2.1 for sigle trait model
``` r
df1=subset(df,Spacing==3)
asreml.batch(data=df1,factorN=1:5,traitN=c(9:13),
             FMod=y~1+Rep+Plot,RMod=~Fam,
             pformula=h2 ~ 4 * V1/(V1+V2))
```
#### exmaple 2.2 for us model
``` r
asreml.batch(data=df1,factorN=1:5,traitN=c(10:13),
             FMod=cbind(y1,y2)~trait+trait:Rep,
             RMod=~us(trait):Fam,
             EMod=~units:us(trait),
             mulT=TRUE,mulN=2,mulR=TRUE,corMout=T,
             pformula=r.g ~ V2/sqrt(V1*V3),
             pformula1=h2.A ~ 4*V1/(V1+V5),
             pformula2=h2.B ~ 4*V3/sqrt(V3+V7))
```

#### exmaple 2.3 for corr model
``` r
asreml.batch(data=df1,factorN=1:5,traitN=c(10:13),
             FMod=cbind(y1,y2,y3)~trait+trait:Rep,
             RMod=~corgh(trait):Fam,
             EMod=~units:us(trait), maxit=30,
             mulT=TRUE,mulN=3,mulR=TRUE,corM=TRUE)
```
### example II for dataset with pedigree
``` r
ped<-df2[,1:3]  
pedinv<-asreml.Ainverse(ped)$ginv
df2a=subset(df2,Spacing==3)
```
#### example 2.4 sigle trait model
``` r
asreml.batch(data=df2a,factorN=1:6,traitN=c(7:14),
             FMod=y~1+Rep,RMod=~ped(TreeID),
             ped=T,pedinv=pedinv,                                   
             ginverse=list(TreeID=pedinv),
             pformula=h2 ~ V1/(V1+V2))
```

#### exmaple 2.5 us model 
``` r
asreml.batch(data=df2a,factorN=1:6,traitN=c(10:14),
                    FMod=cbind(y1,y2)~trait+trait:Rep,
                    RMod=~us(trait):ped(TreeID),
                    EMod=~units:us(trait),maxit=40,
                    mulT=TRUE,mulN=2,mulR=TRUE,corMout=F,
                    ped=T,pedinv=pedinv,
                    ginverse=list(TreeID=pedinv),
                    pformula=r.g ~ V2/sqrt(V1*V3),
                    pformula1=h2.A ~ V1/(V1+V5),
                    pformula2=h2.B ~ V3/(V3+V7))
```

#### exmaple 2.6  corr model
``` r
asreml.batch(data=df2a,factorN=1:6,traitN=c(10:14),
             FMod=cbind(y1,y2)~trait+trait:Rep,
             RMod=~corgh(trait):ped(TreeID),
             EMod=~units:us(trait),maxit=40,
             mulT=TRUE,mulN=2,corM=TRUE,
             ped=T,pedinv=pedinv,
             ginverse=list(TreeID=pedinv)) 
```
## function 3 model.comp(): model comparison
``` r
fm1<-asreml(h5~ 1+Rep,random=~ Fam, 
            subset=Spacing=='3',data=df,maxit=40)
            
fm1a<-asreml(cbind(dj,h5)~ trait+trait:Rep, 
            random=~ us(trait):Fam, rcov=~units:us(trait),
            subset=Spacing=='3',data=df,maxit=40)
fm1b<-asreml(cbind(dj,h5)~ trait+trait:Rep, 
            random=~ diag(trait):Fam, rcov=~units:us(trait),
            subset=Spacing=='3',data=df,maxit=40)

fm3a<-asreml(cbind(dj,h3,h5)~ trait+trait:Rep, 
            random=~ diag(trait):Fam, rcov=~units:us(trait),
            subset=Spacing=='3',data=df,maxit=40)
            
fm3b<-asreml(cbind(dj,h3,h5)~ trait+trait:Rep, 
            random=~ diag(trait):Fam, rcov=~units:us(trait),
            subset=Spacing=='3',data=df,maxit=40)
```
comparison between two models:
``` r
model.comp(m1=fm1a,m2=fm1b)
model.comp(m1=fm1a,m2=fm1b,LRT=TRUE)
model.comp(m1=fm1a,m2=fm1b,LRT=TRUE,rdDF=TRUE)
```
comparison among more than two models:
``` r
model.comp(Nml=c(fm3a,fm3b,fm1a,fm1b,fm1),mulM=TRUE)
model.comp(Nml=c(fm3a,fm3b,fm1a,fm1b,fm1),mulM=TRUE,LRT=TRUE)
```

## function 4 met.corr() etc: used for multi-environment trials
``` r
##  met.plot(): plots MET data
##  met.corr(): calculate var/cov/corr from asreml MET factor analytic results
##  met.biplot(): biplots MET factor analytic results from asreml 

data(MET)

##  plot MET data -- example 1
# variable order: genotype,yield,site,row,col
MET2<-MET[,c(1,9,2,4:5)] 
met.plot(MET2)  

## plot MET data -- example 2
MET3<-MET[,c(1,9,2,4:7)] # add variable order on MET2: Rep, Block
met.plot(MET3,"My met trials") 

MET$yield<-0.01*MET$yield
#summary(MET$yield)

met1.asr<-asreml(yield~Loc, random=~ at(Loc):Rep + Genotype:fa(Loc,2), 
                rcov=~ at(Loc):units, 
                data=MET, maxiter=50)

met2.asr<-asreml(yield~Loc, random=~ Genotype:fa(Loc,2), 
                rcov=~ at(Loc):ar1(Col):ar1(Row), 
                data=MET, maxiter=50)

met3.asr<-asreml(yield~Loc, random=~ Genotype:fa(Loc,3), 
                rcov=~ at(Loc):ar1(Col):ar1(Row), 
                data=MET, maxiter=50,trace=F)

## count var/cov/corr matrix, etc.
met.corr(met1.asr, site=MET$Loc, faN=2, kn=2)
met.corr(met2.asr, site=MET$Loc, faN=2, kn=2)
met.corr(met3.asr, site=MET$Loc, faN=3, kn=2) 

## biplot asreml-met results

met.biplot(met2.asr, siteN=6, VarietyN=36, faN=2)
met.biplot(met3.asr, siteN=6, VarietyN=36, faN=3)
met.biplot(met2.asr, siteN=6, VarietyN=36, faN=2, dSco.u=1.8, dLam.u=0.8)
met.biplot(met2.asr, siteN=nlevels(MET$Loc), VarietyN=nlevels(MET$Genotype), faN=2) 
# dLam.u -- least distance from center
# dSco.u -- least score of Variety breeding value
# if can not draw fig 3, try multiplying or being devided by 10 for aim trait data.

``` 

### More details will be updated in the future.
