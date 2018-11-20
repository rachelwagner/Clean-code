# Clean-code
/* Clean code */

/* Loading the data. Excluding any empty or missing fields for reviews */
insheet using /mnt/commons/rachel/100kSamplOfAllSeed4694329905allBooksReadMin1revV1.csv, clear names
keep if rev~=""
keep if rev~="."
keep if rev~="..."

//Loop and create individual string words (str_'word') for empath dictionary
gen count=wordcount(rev) //do this first to generate rev count 
foreach word in "felt" "feel" "both" "together" "close" "unite" "believe" "accept" "fathom" "grasp" "hope" "tender" "humanity" "humane" "sorrow" "heartfelt" "warmhearted" "sincere" "emotional" "percept" "understand" "caring" "care" "compassion" "compassionate" "giving" "considerate" "concern" "friend" "admire" "realize" "experience" "warmth" "warm" "kind" "sympath" "empath" "gentle" "sensitive" "sense" "affection" "affectionate" "generous" "thought" "thoughtful" "romantic" "romance" "sentimental" "sentiment" "forgiving" "critical" "mean" "arrogant" "selfish" "safe" "peace" "protect" "shield" "shelter" "amity" "secure" "secur" "security" "benefit" "defen" "guard" "preserve" "harm" "suffer" "fight" "violen" "hurt" "kill" "endanger" "cruel" "brutal" "abuse" "damag" "ruin" "ravage" "detriment" "crush" "annihilate" "destroy" "stomp" "abandon" "spurn" "impair" "exploit" "wound" "think" "understood" "perceive" "viewpoint" "outlook" "standpoint" "stance" "perspective" "affinity" "appreci" "comprehend" {
di "`word'"
egen str_`word'= nss(rev) , find(`word') insensitive //3,226
//gen p_`word'=(str_`word'/count)*100
}

egen tot=rowtotal(str_*) // gen total of all E words 
gen EProp=(tot/count)*100  //putting total empathy words over total words

//generate date 
gen year=substr(datadd, -5,4) 
gen daymo=substr(datadd, 5,6) 
gen mo1=substr(daymo, 1,3)
gen day1=substr(daymo, -2,2)
l datadd year daymo mo1 day1 in 1/10
// Replacing month words with month numbers ("Jan" to "1") 
gen mo2=""
replace mo2="01" if mo1=="Jan"
replace mo2="02" if mo1=="Feb"
replace mo2="03" if mo1=="Mar"
replace mo2="04" if mo1=="Apr"
replace mo2="05" if mo1=="May"
replace mo2="06" if mo1=="Jun"
replace mo2="07" if mo1=="Jul"
replace mo2="08" if mo1=="Aug"
replace mo2="09" if mo1=="Sep"
replace mo2="10" if mo1=="Oct"
replace mo2="11" if mo1=="Nov"
replace mo2="12" if mo1=="Dec"
gen date=year+"-"+mo2+"-"+day1
ta date // it worked!

destring mo2 day1 year, replace
gen da=mdy(mo2, day1, year)
//tsset usrid da
//this makes the date numeric 


/*check if right!! duplicates!!! check with dan!! This is a limitation***
there are people reviewing multi books on te same day. */
sort usrid date
edit usrid datadd date datupd readat 

//generate "timer" or # of times reviewed per day
gen counter=1
bys usrid: egen countSum=total(counter) 
sort usrid date
bys usrid: gen timer=_n

sort usrid da
line EProp da in 1/100, by(usrid)
// if countSum>50 & countSum<500


corr EProp timer  
lowess  EProp revcounter if revcounter<300

sort usrid date
bys usrid:gen revcounter=_n
l usrid date revcounter EProp in 1/1000, sepby(usrid) 
collapse (mean) EProp, by(revcounter) //mean for 1st rev 2nd rev so on...

//trying to collapse by date 
collapse (mean) EProp, by(date) //this works but then cannot graph it

lowess  EProp da if revcounter<300 
lowess EProp revcounter


line EProp da
