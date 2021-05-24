# Breaking Google’s Image-based reCaptcha v2

I dette projekt har vi valgt at vi vil forsøge at bryde Google’s reCAPTCHA v2 med et neuralt netværk.
reCAPTCHA tester den menneskelige evne til at genkende objekter på billeder, derfor er det en oplagt opgave at forsøge at løse med et neuralt netværk.
Mange virksomheder er for længst skiftet væk fra reCAPTCHA v2 til andre bedre alternativer, men man støder stadig ofte på denne reCAPTCHA.



## Collect Data
Da vi skulle finde et datasæt til at træne vores model, prøvede vi at google “recaptcha datasæt”, for at se om der allerede var nogen der havde lavet et. 3. link førte til en github repository kaldet “recaptcha dataset” (https://github.com/brian-the-dev/recaptcha-dataset).
Efter et nærmere kig på billederne i datasættet, viste det sig at være et rigtig godt udgangspunkt for vores projekt.
Datasættet indeholder mere end 10.000 billeder, fordelt på 11 kategorier såsom “Bicycle”, “Crosswalk” og “Traffic Light”, som er det man ofte skal identificere i googles recaptcha.
Desuden er billederne allerede skaleret ned og beskåret til et 1:1 format, på samme måde som i recaptchaen. Datasættet er også frigivet under en MIT licens, så vi uden bekymringer kan gøre brug af det.

En potentiel udfordring ved datasættet er dog, at flere af billederne passer ind i flere kategorier.
Fx er der mange af billederne af “Traffic Lights” som også indeholder en “Crosswalk”.
Datasættet er dog lavet sådan, at hvert billede kun er i en kategori, og tager ikke højde for om det passer ind flere steder.

Efter at have downloadet datasættet valgte vi midlertidigt at trimme størstedelen af billederne væk, så der kun var en lille mængde billeder i hver kategori.
Det gjorde vi for at gøre det hurtigere at træne modellen under de indledende kode-iterationer.
Først da vi var overbeviste om at vi havde styr på koden, trænede vi modellen op imod det fulde datasæt.



## Prepare the data
###Opsætning af datablock
Forberedelsen af data’en sker i vores datablock.
Vores block er en ImageBlock som er MultiCategoryBlock. Det vil altså sige at vi vil have modellen har mulighed for at kunne genkende mere end et label på billederne.

get_items bliver hentet gennem fastai’s get_image_files metode.
get_y bliver sat gennem egen metode get_y som sætter parent_label til billedet.
splitter bliver sat efter default parametre hvor der er en valid_pct på 0.2 og seed er 42.
item_tfms sættes til 128 og min_scale til 0.35. Dette gør at billederne har en minimumsstørrelse på 128 pixels med en scale på minimum 0.35

Når modellens datablock bliver instansieret sættes der ikke en batch_tfms.
I dette tilfælde betyder det at fastai automatisk får lov til at normalisere data’en udfra en enkelt batch af data’en.

###Dårlige billeder
I det datasæt som der er blevet fundet er der ikke blevet fjernet dårlige billeder.
Dette skyldes bl.a. at en reCaptcha er sat op til at kunne have dårlige billeder, hvor det som der bliver spurgt efter ikke nødvendigvis er i fokus eller god kvalitet.
###Resultat
Som det kan ses i opsætning af datablock’en, så er de fleste værdier “default” værdier, som fastai anbefaler man benytter i første opsætning.
Dette har vi ikke ydeligere modificeret på eftersom at vores baseline model har resulteret i en accuracy på op til 94%.



## Choose the model
Et konventionelt neural networks network design har hver enkelte node forbundet til alle noder finding in i alle node i næste lag.
Dette gør at backprobergationen selv skal finde mønstre og segmentere datagen.
Med et Convolutional Neural Networks tvinger man netværket til at kun at kigge på mindre segmenter i stedet for selv at gøre det.
Med Convolutional Neural Networks findes der mange strategier man kan bruge til en neural Network design så som LeNet-5, AlexNet og VGG-16.
I vores projekt har vi brugt den indbygget cnn learner for multi category block.
Vi bruger Multi Category Block da det var et krav men også lader os benytte vores ressourcer bedre da vi kan tjekke hvad Recapture spørg efter samtidig med billederne bliver labeled.

![]({{ site.baseurl }}/images/Screenshot_1.png "")

Mens vi undersøgte hvordan vi optimerer vores model valgte vi at lave et mindre datasæt end havde vi oprindeligt fandt. 
Dette gjorde at vi kunne iterere hurtigere da det tog kortere tid at træne når vi laver ændringer i vores lærings parameter.
Ulemper med denne tilgang er tendenser i vores træning sæt bliver forstærket fx de fleste modeller vi har trænet har ikke haft multi category labels på billederne men derimod kun et label.
Dette gjorde hvis der var en bro og en bil på samme billede, ville den kun vælge den ene frem for begge. Mere om dette under Prediction afsnittet.
Vi er endt med en model det svare med Multi Category Block og bruger fastai’s indbyggede cnn lerner, og for hurtig iteration har vi et mindre datasæt.

![]({{ site.baseurl }}/images/Screenshot_2.png "")



## Train your machine model
Med dataene og fremgangsmåden redegjort for, er vores projekt i træningsfasen.
Her følger vi træningen igennem vores udvalgte data, og validerer gennem fasens epochs hvilke antagelser som modellen laver, og hvilke der ender ud med at være korrekte.

I datasamlingen fortalte vi om vores opdeling af vores datagrundlag og en test sample, som er trimmet ned for at tillade at den indledende træning forløber hurtigt.
Dette gør at vi kan opbygge vores optimale indstillinger for træningen og klargøre til det komplette datagrundlag.

Vi kan forsøge os med at tilføje flere kørsler i form af Epochs for at finde ud af hvilken kvantitet af træning der giver os umiddelbart den bedste nøjagtighed.
I vores trimmede dataset kører Epochs hurtigt, og giver os et indblik i hvilke resultater vi kan forvente.

I træningen eksperimenterer vi også med forskellige Resnets for at gøre træningen hurtigere, hvilket baner vejen for en hurtig opstart i de første Epochs, som så tilføjer de mest komplekse layers tilbage i de senere Epochs for at opbygge et grundlag for modellen og validere den til sidst.
Vi valgte at gå med Resnet50, da vi havde størst accuracy med testen undervejs og havde mest erfaring med denne mængde.

Det giver også mening at effektuere træningen ved hjælp af CNN learner’ens Learning Rate Find metode.
Dette giver et indblik i hvordan CNN bedst kan lærer ud fra vores data på den mest ressourceeffektive måde.



## Evaluation
[Baseline model notebook]({{ site.baseurl }}/2021/05/25/Captcha-Baseline-Model.html/)

For ikke at skulle træne modellen på alle 10.000 billeder hver gang da det taget lang tid har vi lavet et datasæt der tager 20 billeder fra hver kategori.
Dette datasæt har vi efterfølgende brugt til vores aktive udvikling.
Når vi har haft en ny model iteration vi gerne ville teste helt, har vi skiftet tilbage til det oprindelige datasæt med 10.000 billeder.
Vi har oplevet at dette lille datasæt har været tilstrækkeligt til at give en indikation af hvor godt modellen er.

I vores DataBlock har vi benyttet en RandomSplitter der udvælger 0.2 procent af billederne som vores valideringssæt.
Billederne bliver også alle kørt igennem en RandomResizedCrop der gør at vi får mere variation i vores data.

Vi har benyttet den indbyggede leaning rate finder (Lr_find()) til at finde den optimale learning rate for vores learner.
Lr_find() metoden køre et par iterationer med en meget lille learning rate.
For hvert mini-batch bliver der skruet en lille smule op for learning raten indtil den til sidst er meget høj.
Iterationernes loss bliver sammenlignet med den valgte learning rate og den bedste bliver valgt.

Da vi trænede denne model første gang med ovenstående udgangspunkt, blev vi meget overraskede over at vores model opnåede en accuracy på 94%.
Da vi testede modellen med et billede den aldrig have set før blev vi lige så overraskede.
Modellen var 99.6% sikker på at der er bro i det billede, men den var kun 0.07% sikker på at der var en bil også selv om det er meget tydeligt at se at der både er biler og en bro.
Dette betyder at vi har lavet en model der er rigtig god til at finde en enkelt ting i et billede men altså ikke flere ting som vores model skal kunne.

Vi har alligevel valgt at beholde denne model som vores baseline model på trods af dette da den giver et godt udgangspunkt vi kan teste vores fremtidige iterationer op imod.



## Tuning / Optimisation
[MixUp notebook]({{ site.baseurl }}/2021/05/25/Captcha-MixUp.html/)  
[Label Smoothing v1 notebook]({{ site.baseurl }}/2021/05/25/Captcha-LabelSmoothing.html/)  
[Label Smoothing v2 notebook]({{ site.baseurl }}/2021/05/25/Captcha-Label-Smoothing-v2.html/)  
[Label Correction notebook]({{ site.baseurl }}/2021/05/25/Captcha-Label-Correction.html/) 

Med vores nyoprettet baseline model, justerer vi vores forskellige parameter for at opnå det bedste resultat.
For at sikre imod at vores model blev for sikker og “overfitted”, brugte vi Label Smoothing undervejs i træningen for at gøre modellen mere modtagelig over for et mere nuanceret gæt i sidste ende, sådan at modellen kan være bedre til at gætte på andre mulige resultater i de billeder, som vi skal genkende.
Dette er rigtig nyttigt, til billeder som indeholder fx biler og broer, hvor at begge klassifikationer er korrekte, og gør at vi kan validere om modellen tror at en bestemt kategori findes på billedet.
Når reCAPTCHA så spørger efter alle bjergene på en række billeder, så er det ikke et problem at der er en bil på nogle af billederne.
Vores model kan stadig vælge at kigge efter den specifikke kategori, beregner hvorvidt der er en sandsynlighed for at et billede indeholder et bjerg og kan på den måde stadig svare rigtig.
Dette kræver dog bare at vi specificerer et threshold for hvor lav sandsynlighed der skulle accepteres.

Til sidst i vores forsøg på at optimere vores model, tilføjede vi en Mixup Callback metode til vores learner for at give bedre nøjagtighed på vores billedgenkendelse.
Med Mixup kan vi blande vores billeder sammen for at billederne skal genkendes ind over hinanden i et forsøg på at gøre learneren åben over for tvetydighed i vores data.
Når vores learner støder på et billede der er opbygget af 20 % biler og 80 % lyskryds, vil vores forventning være at den bliver bedre til at opdage flere kategorier i hvert billede, hvilket i sidste ende vil forbedre vores metode.
Det kan desuden også diskuteres over at vores model ikke skal være perfekt, da et menneske ligeså godt kan tage fejl af et objekt i et billede, hvilket Mixup også kunne tilføje til vores model.



## Prediction
Efter som vi arbejder med en Multi Category block har vi haft svært ved at bruge de mange af de redskaber vi er blevet undervist i så som confusion matrixen.
Da vores billeder primært kun har haft et labelt på har vi under det meste af udviklen haft stor bios i visse tilfælde.
Fx har der ofte været biler på billeder med bruger dette har fået AI’en til at være sikker på bruger frem for biler når der de har optrådt i samme billede.
Tobias lavede en csv fil på vores lille datasæt for at migrere dette problem.
Vi har prøvet at køre vores model mod en “rigtig” recapture billeder fra https://www.google.com/recaptcha/api2/demo.
I stedet for at læse html siden med Python downloade vi billedet og manuelt splitte det i de 9 billeder, hvor vi efterfølgende putte det ind i vores AI.
Vi endte med det nedenstående resultat, den fangede næsten alle cyklerne den eneste den mistede var 2,3.
Dette kunne skyldes at vi ikke bruger hele vore dataset da vi trænede denne model men i stedet vores mindre træningssæt.

![]({{ site.baseurl }}/images/Screenshot_3.png "")  
Image: 1, 1.jpg Prediction: ['Bicycle']; Bicycle Probability: 0.9659770131111145  
Image: 1, 2.jpg Prediction: ['Bicycle']; Bicycle Probability: 0.9705190062522888  
Image: 1, 3.jpg Prediction: ['Bridge']; Bicycle Probability: 4.990302880554836e-18  

![]({{ site.baseurl }}/images/Screenshot_4.png "")  
Image: 2, 1.jpg Prediction: ['Palm']; Bicycle Probability: 0.044423192739486694  
Image: 2, 2.jpg Prediction: []; Bicycle Probability: 0.4902642071247101  
Image: 2, 3.jpg Prediction: []; Bicycle Probability: 0.20381507277488708  

![]({{ site.baseurl }}/images/Screenshot_5.png "")  
Image: 3, 1.jpg Prediction: ['Palm']; Bicycle Probability: 0.007280856370925903  
Image: 3, 2.jpg Prediction: ['Crosswalk']; Bicycle Probability: 0.00545891746878624  
Image: 3, 3.jpg Prediction: ['Chimney']; Bicycle Probability: 0.003270353190600872  

Vi prøvede igen men en model der var trænet på hele vores datasæt og det så meget mere lovende ud.

![]({{ site.baseurl }}/images/Screenshot_3.png "")  
Image: 1, 1.jpg Prediction: ['Bicycle']; Bicycle Probability: 0.9953802824020386  
Image: 1, 2.jpg Prediction: ['Bicycle']; Bicycle Probability: 0.9892680048942566  
Image: 1, 3.jpg Prediction: ['Bridge']; Bicycle Probability: 0.0003413711965549737  

![]({{ site.baseurl }}/images/Screenshot_4.png "")  
Image: 2, 1.jpg Prediction: ['Palm']; Bicycle Probability: 0.0001781134633347392  
Image: 2, 2.jpg Prediction: ['Car']; Bicycle Probability: 0.003861015196889639  
Image: 2, 3.jpg Prediction: ['Bicycle']; Bicycle Probability: 0.8249829411506653  

![]({{ site.baseurl }}/images/Screenshot_5.png "")
Image: 3, 1.jpg Prediction: ['Car']; Bicycle Probability: 0.0001316472189500928  
Image: 3, 2.jpg Prediction: ['Crosswalk']; Bicycle Probability: 0.0038241292349994183  
Image: 3, 3.jpg Prediction: ['Other']; Bicycle Probability: 0.0007099288050085306  



## Neurale netværk
### Stochastic Gradient Descent
For at kunne træne vores model så den kan blive bedre til at lave forudsigelser på vores data, skal vi bruge en måde at ændre vores weights.
Dette kan gøres på flere måder men en meget brugt måde er Gradient Descent eller nærmere Stochastic Gradient Descent.
Gradient Descent og Stochastic Gradient Descent gør det samme og er identiske på nær at man i Stochastic Gradient Descent tager lidt data og køre igennem i stedet for alt data som i Gradient Descent.
Dette gør også at Stochastic Gradient Descent er meget mere egnet til arbejde med BIG DATA da du ikke skal lave lige så mange udregnigner for hver epoch.

Man kan forklare hvad Gradient Descent går ud på ved at forestille sig at man skal finde den korteste ved ned af et bjerg.
Hvis man bruger traditionel Gradient Descent til at finde den korteste vej, udregner man den optimale rute på en gang ved at bruge data om alle de forskellige veje der er på bjerget.
Denne løsning giver med garanti den korteste vej ned af bjerget, men det bliver også sværere og sværere at lave udregningen jo mere data man har om bjerget.
Hvis man derimod bruger Stochastic Gradient Descent til at finde vej ned af bjerget, vil man gå ud og kigge på de mulige veje man kan tage på nuværende tidspunkt.
Man går så ned ad den vej der fører en længst ned at bjerget.
Hvis vejen er meget stejl går man lige lidt længer af denne vej inden man igen kigger efter en ny vej der fører en længere og længere ned af bjerget.
Det gør man igen og igen til man har nået bunden.

Hvis vi prøver helt simpelt at overføre dette til vores neurale netværk vil det altså sige at vi tager en lille del af vores data og sender det igennem vores neurale netværk.
Vi udregner gradienten for den data og bruger den til at opdatere modellens weights.
Vi gør så det hele igen med en anden lille del af vores data. Denne proces kan illustreres på følgende måde:

![]({{ site.baseurl }}/images/Screenshot_6.png "")

1. Init: Da vi skal have et udgangspunkt for vores weights bliver de i dette step initialiseret til tilfældige værdiger.
1. Predict: I dette step bruger vi de tilfældige weights til at få en prediction ved at køre en lille del af dataen igennem det neurale netværk.
1. Loss: I dette step bruger vi en loss function til at måle modellens effektivitet med de nuværende weights.
1. Gradient: I dette step udregner vi vores gradient som fortæller os hvordan ændringen af en weight vil påvirke vores loss.
1. Step: I dette step ændrer vi værdierne af vores weights på baggrund af den udregnede gradient. Hvis der stadig er flere epochs tilbage, går vi tilbage til step 2 og gentager de efterfølgende step.
1. Stop: Når der ikke er flere epochs går vi ikke tilbage til step 2 men stopper derimod bare.
      
Det vigtige i denne proces er altså step 4 hvor vi udregner vores gradient.
Gradient er bare et fancy ord for derivative der kommer fra calculus.
Så når vi siger at vi udregner vores gradient udregner vi i realiteten derivative for vores loss function.
Teknisk fortalt er derivative en funktion der beregner slope for en funktion ved en given værdi.
Mere simpelt fortelt fortæller derivative os hvor stejl bjerget er på en given vej.



## Optimering
### Smoothing (Label Smoothing)

Når vores model bliver trænet så er vores mål i teorien at få et resultat der bliver 1.
Dette er resultatet på modellens “gæt” også selvom den ikke er 100% sikker.
Dette betyder også at de resterende kategorier bliver 0.
Denne tilgang giver tendens til overfitting, og en model som ikke giver betydelig respons på dens gæt.

For at undgå dette så kan man under opsætningen af sin model have en loss function som benytter sig af Label Smoothing.
Label smoothing sørger for at modellen kommer med et mere generaliserende gæt, ved at erstatte alle 1’erne med et tal mindre end 1 og alle 0’erne med (𝝐 / N), hvor 𝝐 (epsilon) er et parameter oftest 0,1 og N er antallet af klasser.

Når vi træner modellen vil vi stadig gerne have at alle labels til sammen giver resultatet 1, derfor erstatter vi 1 (det label som der bliver gættet på) med følgende formel: (1-𝝐+𝝐/N).

#### This maximum is not achievable for finite  zk  but is approached if  zy≫zk  for all  k≠y
Hvis man går over i den teoretiske forståelse af label smoothing, så er det en matematisk algoritme (stående ovenfor) som beviser at maximum ikke er opnåeligt.

Hvis man deler algoritmen op, så er y vores target og zy den activation som er tilhørende det target.
For at komme tæt på 1, så skal zy være større end ved alle andre “gæt”.
Dybere i algoritmen bliver det forklaret, at hvis modellen lærer at tildele fuld sandsynlighed på labellet ved hver træning, så er den ikke garanteret at kunne generalisere.
Det som det betyder er at hvis zy har en høj/stor værdi, så skal der benyttes større weights og activations gennem modellen.
Dette kan resultere i at små ændringen i input (f.eks. ændring i en pixel) kan give en helt anden probability når der bliver gættet.

Hvis man holder algoritmen op sammen med Bounded Gradient, så reducerer det modellens evne til at kunne tilpasse sig.

#### Sidebar
Benyttelse af Label smoothing ses ofte kun ved Single-Label.
Selve matematikken bag Label smoothing er i første omgang tilegnet Single-Label, og man vil derfor skulle have en forståelse for at kunne ændre i selve algoritmens matematik for at kunne benytte Label smoothing ved Multi-Label.

Det har ikke været muligt at finde nogle endnu som har knækket den matematiske kode til at benytte Label smoothing på Multi-Label.
Den offentlige debat lyder i hvert fald på at det i det nuværende omfang ikke er muligt at benytte Label smoothing på Multi-Label modeller.



## Fastai
### DataLoaders
DataLoaders er en meget simpel klasse, som bare wrapper/indeholder en række DataLoader objekter.
Det er dog den som står for at levere data til modellen, og det er derfor en meget central del af FastAI.

Der er flere måder man kan lave ens DataLoaders objekt på.
FastAI indeholder en række såkaldte “factory methods”, som returnerer DataLoaders med prædefinerede opsætninger.
Det er en smart måde at spare tid på, hvis det man laver er så “almindeligt” at der allerede er en opsætning, som passer. 
Alternativt kan man lave en custom DataLoaders. For at gøre det, bruges data block API’en.

Når man laver en custom DataLoaders skal følgende angives, for at modellen kan trænes:
- Independent variable
- Dependent variable
- Hvordan skal data hentes
- Validation set
- Labels

Derudover kan man angive item transforms, som er noget kode der bliver kørt på hver entry i datasættet.
På samme måde som med DataLoaders har FastAI en række indbyggede transforms, til de mest gængse operationer, men det er også muligt at kode sin egen.
De indbyggede transforms inkluderer fx cropping og resizing af billeder.

Som et alternativ til item transforms, kan man bruge batch transforms.
Med en batch transform kører man i stedet noget kode på hele batchen i stedet for hvert individuelt billede, hvilket kan være hurtigere at udføre. 
transform kan fx bruges i forbindelse med data augmentation af billeder, efter de er blevet croppet og/eller resizet til alle at være den samme størrelse.
