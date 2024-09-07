+++
title = "Issuing contract"
weight = 30
[extra]
bg-color = "red"
anchor = "issue"
+++

To issue a contract you need to write a contract declaration file. Contract
declarations can be done in multiple languages, including Rust, YAML,
and future [Contractum](https://www.contractum.org). Here we will stick with
a YAML format as most easy-to-use and human-readable format, writing which
doesn't require any developer skills:

```yaml
interface: RGB20Fixed

globals:
  spec:
    ticker: DBG
    name: Debug asset
    details: "Pay attention: the asset has no value"
    precision: 2
  terms:
    text: >
      SUBJECT TO, AND WITHOUT IN ANY WAY LIMITING, THE REPRESENTATIONS AND WARRANTIES OF ANY SELLER 
      EXPRESSLY SET FORTH IN THIS AGREEMENT OR ANY OTHER EXPRESS OBLIGATION OF SELLERS PURSUANT TO THE
      TERMS HEREOF, AND ACKNOWLEDGING THE PRIOR USE OF THE PROPERTY AND PURCHASER’S OPPORTUNITY 
      TO INSPECT THE PROPERTY, PURCHASER AGREES TO PURCHASE THE PROPERTY “AS IS”, “WHERE IS”, 
      WITH ALL FAULTS AND CONDITIONS THEREON. ANY WRITTEN OR ORAL INFORMATION, REPORTS, STATEMENTS, 
      DOCUMENTS OR RECORDS CONCERNING THE PROPERTY PROVIDED OR MADE AVAILABLE TO PURCHASER, ITS AGENTS
      OR CONSTITUENTS BY ANY SELLER, ANY SELLER’S AGENTS, EMPLOYEES OR THIRD PARTIES REPRESENTING OR
      PURPORTING TO REPRESENT ANY SELLER, SHALL NOT BE REPRESENTATIONS OR WARRANTIES, UNLESS
      SPECIFICALLY SET FORTH HEREIN. IN PURCHASING THE PROPERTY OR TAKING OTHER ACTION HEREUNDER,
      PURCHASER HAS NOT AND SHALL NOT RELY ON ANY SUCH DISCLOSURES, BUT RATHER, PURCHASER SHALL RELY
      ONLY ON PURCHASER’S OWN INSPECTION OF THE PROPERTY AND THE REPRESENTATIONS AND WARRANTIES 
      HEREIN. PURCHASER ACKNOWLEDGES THAT THE PURCHASE PRICE REFLECTS AND TAKES INTO ACCOUNT THAT THE
      PROPERTY IS BEING SOLD “AS IS”.
    media: ~
  issuedSupply: 100000000

assignments:
  assetOwner:
    seal: tapret1st:b449f7eaa3f98c145b27ad0eeb7b5679ceb567faef7a52479bc995792b65f804:1
    amount: 100000000 # this is 1 million (we have two digits for cents)
```

As you can see, contract definition consists of:
* specification of the interface used for constructing the contract and parsing
  provided state
* global state – in this case specifying token nominal information and giving
  legal contract text
* assignments, which assign owned state (issued asset amount) to a single-use seals,
  which are bitcoin UTXO prefixed with a type of seals (`tapret1st` stays for
  "tapret 1st output").

You compile the contract and issue it with the command `rgb issue`. It takes
three arguments:
* the name of a schema which is used as a template for the contract;
* an information about the issuer identity;
* YAML file with the contract text we have written above.

Schemata are templates written by professional developers, which hides the
contract complexity from the issuers. You may look for them at <https://rgbex.io>,
or just [download a kit](kit) with the one containing the schema we will be
using today.

```sh
$ rgb import ~/Downlads/NonInflatibleAsset.rgb
```

You will see a number of messages on a successful import, which contains
ids of the components you have added to the RGB stock.

```
Importing kit rgb:kit:$Rniyeda-KT4Rnce-aN9gKgA-xkhhVMC-y4SEO8L-KQd9DYI:
- interface RGB20Fixed $iUnO9aO-1xhqUd6-1Jm5S5!-wM3ngby-5GVEylQ-ZTAMYDk#tornado-pioneer-bucket
- schema NonInflatableAsset RDYhMTR!9gv8Y2GLv9UNBEK1hcrCmdLDFk9Qd5fnO8k#brave-dinner-banana
- implementation of RGB20Fixed for NonInflatableAsset
- script library alu:q$CZ0ovt-UN9eBlc-VMn86mz-Kfd3ywu-f7$9jTB-k6A8tiY#japan-nylon-center
- strict types: 89 definitions
Kit is imported
```

Here we can read that the id of the schema is `RDYhMTR!9gv8Y2GLv9UNBEK1hcrCmdLDFk9Qd5fnO8k`,
please copy it. Now, we can finally issue the contract:

```sh
$ rgb issue 'RDYhMTR!9gv8Y2GLv9UNBEK1hcrCmdLDFk9Qd5fnO8k' ssi:anonymous examples/rgb20-demo.yaml
A new contract rgb:D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0 is issued and added to the stash.
Use `export` command to export the contract.
```

Please pay attention that the schema id is put into a single quotes: RGB ids
may contain characters like `$` and `!` which has a special meaning in the
command shell; if no quotes - or double quotes are used, you will probably
get an error.

One may also develop a custom, more advanced schema, providing such facilities
as secondary issue or DeFi functionality. To do that, please check
[programming RGB](/program) on this website.

The issued contract was added to the stash. You may see it by running

```sh
$ rgb contracts

rgb:D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0	bitcoin            	2024-09-07	rgb:sch:RDYhMTR!9gv8Y2GLv9UNBEK1hcrCmdLDFk9Qd5fnO8k#brave-dinner-banana         
  Developer: ssi:anonymous
```

To distribute contract to other users you have to export it in a form of
**contract consignment**. This consignment may be a binary file - or 
ASCII/Base64 armored text, such that it can be sent via messengers or by an
e-mail (or even put into a dynamic QR code).

```sh
$ rgb export 'D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0'
```

By default, this prints the contract in ASCII armored form to the terminal.

```
-----BEGIN RGB CONSIGNMENT-----
Id: rgb:csg:4!k8VkFo-3Kc1KIw-08wnjOw-u$7Qg89-Nwu3Kg5-oq!MoWM#sister-chance-imagine
Version: 2
Type: contract
Contract: rgb:D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0
Schema: rgb:sch:RDYhMTR!9gv8Y2GLv9UNBEK1hcrCmdLDFk9Qd5fnO8k#brave-dinner-banana
Interface: RGB20Fixed
Check-SHA256: d1938b9bde125082e7078d053e604a73200e78f456c4dd663ff274b93bfd4480

0ssI2002ZbAu%+5_6z)DVT-@j4Fp29h04O2(!&-{P<NN-JIMf4lH6th0000D0CRI`I$>^aZh38Qb#nm#
0iX{an?byYI6I(On(v&Y_J51|kNPSBNMBNFlDt%HFCVc01JDNn05|{xL_$XkL}g-iXCPs7b7gb^B~W2`
AYpWLWo~q7Z*DpubZBKDVRLh3bRcM9b0BVSAa-GFb!7t42LS+;1d;?(RYFQdLsTGCPb?roPDCJANmNKr
Ra78JP9Q-}Ss+(ISs+YFO-WQqPDd;tR7gc2QbkZwMN>siR6$fpPfk-HK~6*<S3y!zK~7XjMN=S8Mj$~>
Ss+tIOiV>mARt9pP*O!xQ%qSPQ$<uDMo&^yNFYg0AXG?6Qy@V{Qbk2gMNU*8Pf{R3PFWyNR7gcqAVpYE
Qbki!AWuR}Nk>6cNl#87Peve9MNCXZQd1yMRZ>$`K~7X4R8JsONJSu2MN&;uAV@`0MNdX7AVE$<AVEV*
PES`%MMOtQPDdbANJSt}Qb|uzAXQUEAWudhR7gc2P*P7&MN(8*AVE$<AW&6OLr6hWMN;B`nNuK7P*6`&
R8>w%R9PS(R8JsDPE$}tLsTGCNJSt}QcqAtQdC(iAW&6OLr6hWMN%L^M^Z&aQy^4NAW&6OLr6hWMIcm2
MIca8Pf$ftR9PV6fSf^7AW2i=fSoKL;((l2NJUabAW2i=fSoKLAXiCLNFYH>Odv)<RZLV<AVE$<AVW`1
L`hUhPfk-HR7gcqMNdvHAVE%9AXidJR8&PyAWu>tPf|flAW2R}Pf|@mR7p=xEFe-vP)|}+Q!F4;R6$fl
O+`*rQ!F4LL{CFiO+`*rQy@=LAW}s`Pf|ovAVW`1Lq$?fNlr%~R7gc2P*P7&MN(8*AW%|IR!KxfL?BO6
AWcC;MIb>|K}k$OLQF*<R8JsKRZ>GpK~qIiEFei#Qy@V{MNU*xAWu>tLr+dqR7q4-MNU*xAVOInK~7m9
Q$<WnMN%vvK~7m9Q$<WnMN;B`nNuJ^M@3FlQ!F4wO;AivSw%%tAWu>tR7gouL?BQ>QdCJrQy@}BP*O!x
MNU*nPDdb5QXo)OQczD)R7p-pAXHBvQbkZwMN>siR3Jf4Ss+tIOiV>mEFe=zK}<{_PES-ILPa1_MNm>j
Q$<cxK~zakPE#OHQXp4BQc^)qR7pisEFe`*Ohr>uAX899LrF$SLqSYTSs+tIR3JuAQdCGFNJUabNlq>x
NlqY8RZ>GpK~qUiM<7&4MIca8Pf$ftR9PTTQXo`8OG!>gAWu|CMN%L^LsUsmP9R7{QbkoxL`708AW&6O
Lr6hWMN%M0K~o@3PgEd5PDCJ6NI^_YAWlzIAW}t4Ss+hNAVE%9AX8OCNFYQ>Q$tKoQ&mz$Q!F4tRa78S
K~zXZQY;`)RZ>GpK~qIiAX7*|OiUnBMNC;BPfko(AWu#pP*qYxNI_FYQsRJ_Qy@=QP9RB6Q&2@iR7p=x
AWudhR7gc2P*P7&MN(8*AVE$<AXG?2AW}t8QbkimPE<iuNl#8wAVE$<AXh<BQbA5sNkvm2AV@`0MM+LB
AW&6OLr6hWMN%L^LrYFiS4>4jM@3U0R7gQoAXG?2AW&6OLr6hWMIca8Nkc^-Qbk5gMMG3mAVE$<AXGt1
MN=S2PE=1IK|@1NRZdhOR7gQoAXG?2AW%|IP(@NySs+PMAVNh+PDdb9PfSE0;((k%Qy@uG;((nl3IN&%
0RRX90O9oo000000iX{70RRI40R;GED|wa4n@3W5@A_x8&UsdQ>kh3aTNI4>qw4obv;hDB01k2`^Nzob
ln4Of^#uR`004e|etv#_etv#_etv#_etv#_etv#_etv#_etv#_etsUCLA;1KJD^#b@0_Lfe~bE$`YHiP
Us7t4yi{#3AF%)c00000000000000001{4bZb@!tY+-a^Vr*qWb8}^Mkc}T^00000GyrpRX*x_=Q!#aT
EoW*(Ic```MlDZcWpq_lYga8cax-;PLsK>_VNqyvIaf7iEjUU=H+KL7&<6n5{J!HJ@Tgs1mpj@U3yhwA
`^&{wC3iS1tkb=;A&LP3007Yk09%X4R5&sPN*yA;lp<^AQ;QQiCWsumMiT;fc;H-Y_W=L^+6MrLj96u3
I`KP|x6K-jit^gQ+!PC!a#7jT+VjUz9FBwm0004?4*>`O00Ynm0RRC2(FXwl0RY+u0RRC20iX{70RR60
0juzt(u?g--(Ch+6*7N1n=+qwe6YFx|MoP&lb}4dCJ6ul0T3qu00E#60RaF10iX{70RR600juzt(u?g-
-(Ch+6*7N1n=+qwe6YFx|MoP&lb}4dCIA2c00000000010SZz_LNYK$X?SI11XfR6<MVPB{R!&WDGXgX
2u=0cfR=^v4L0bS8HU8WU)OVbJscBHuRauJB?krea%!Hk0(I6sCqhFtSs;lPKAoX!0r&<f`@aL{H!tSG
|BE#(Vo+CtFZ->o;2qj;#=~;t^vidEtAC%IlVX)dgF*wOtts}w(E1kon`A59t%UgjW&i*H0009FX>)UR
Wn@!zaBysS0f>xPWn((=JC(Q18jXtb+QHlu3zu?H+0@$e$59-PgaH5qb8uy20oVM#;~wy+U0;_w+8Yau
o__nw#aAVFI4rEwy|f{U0RaF7bY*gFa{*h6$5c2n1xg(vzLX+s=TnOlIwpu5x<(TMczEDkZ1({G0SaMr
b7gc-cWz~J0ssL4000033~6(7b!B8zb#QQOc>w?c00eVzWn%#V0RRPbWpZtE0RRC20SaMrb7gc-cWz~J
0RaF1009nZb8~fNWKC&vZDDj{XaNXxa$#<BW@T~!000013So0|Wpqz>Ze?--0RR613So0|Wpqz>Ze?--
0RR600S|6(Zbfl*VQfKdZ*^{Ta{&rrb8}^MPj_x*asUAcbaG*Cb7p070uE_&b9H58O=)v&VRU0?WOH?J
aBO)Xb8uy2X=Z6<WFTR4AYmY9Y;R&=Y#?x9a$#*{bY*fNWN&42ZYOjgZDDj{XdrZGWguyDb9H58Aaiwa
aBO)XVQg$~V_|e<WFT~JAarPDAYpTJWpp5KcWz~Ja}REBZbfl*VQfKdZ*^{Tb47G$Wgv5PZ6I%EAaihK
Zge1Fb8}^Mb0B1IWpi#PbRcDMbzy8EbZ;PZXk{RCb!{MTW*}j6b7gdMAZczOZ*_EVb#!wy0CRI`I!szq
F?Dz?XKF+_Zdg!8El*=*bX8SrS1mSjGj&r#Q#LJOQD}2HS2boWI7&q~cL78;Au%+5_6z)DVT-@j4Fp29
h04O2(!&-{P<NN-JIVefCp*@T*BEM1-nAxfQs?Xp-gq0!k(CoEP-QR-U^$SDA7%gm00003&<6x_aAjiv
0002d2L*Ixa&2<}0002m2MlR*b9H58Q+04~Y<U0x0004?4+>#(b7gc-cWz~J00000009su2y}8`ZgXa3
asU7T000624{mR6MR9duY(Z^rb#8QX000000S;+%b9H58O=)v&VRU0?00000GyrpRX*x_=Q!#aTEoW*(
Ic```MlDZcWpq_lYga8cax-;PLsK>_VNqyvIaf7iEjUU=H+KLd000X1U)Cjo-i6E2P9x&mnv%Qki+Oba
;k67*blZ=H=TQh>UMA(m1w0!?L{VGDpk+OvDhHAKF%fNXr25$w;Zs!r0000000007000000000O%am^t
lg}6qop{{FTg974FaQ3n`}K{nn9PGH_DcZ;0agu`_oR6wvcum54rF6FkJew+k!36?Lqfl$`8gF)R2-|n
!`LRkztQP;3W%Qi%!_9htpQ3t>=3qD6)+-@LI6M<uyu|U!T^j90F36+)E=H0H{s0>nH0sEektM3d!V}o
0nEa3l8<>f1KA&4t&LI4m}^5aI1iE}_s79eO?HmEkSbfMtWb&n35^vCNG$%?ywDnvz}K{0G9hl&cB^sg
-3J`2zr)xjz`xPycM6D}`pk=G7OeqFKI{;-SrsrMkU}5;FWB;W7bg;sK59Oe@c3K=fV3eR7p&1RS^QDd
q`TfM1OfmAZf|a7*gwADFAe3iZ1@l19{2t5VaJV^T`{fc?xMUvnKPbj0R(ezZDp`<(~tJj3|i&b2NlXO
R2^DUyWY#wQk^*F-L`Td370(4qMgjGn~{4aFkgwNr28QlFe*-S#jFZ=4d$x=UULI21Z8+*Y#{__VRL9B
24rt+Y+-UF17U4&CIoP7b#p5OWMOk?Edyk4bS?yXWpZyY18;6+F#~jWZ!!gRXmVv`GX!RDb#gQWW@&b1
H3M^Lcs2!dWp-t5Hw9&BXJ~Xd1a4_=WjO_7VRB`3UIuJ$WMOk?UjboZ0b*hSV`BkiWC3Mm0cK_aXJ-Lu
XaQ+y0cvUiYij{)YyoX;0d8&qZ*Ku`Z~<{~0djHyb8`W7bOCjB0d{r)cXt7Jcma8N0eX5rD{{BQuNq?v
w$uLzi?1~hlkP@ao_$9uVE}^UN!R2B0b{Bo6zH)>$g+grvznd|(VVK)`s##^Jh_COk!RL4N<*rD#rE}N
PvxSnUK**8Le1-kltSZ7azFKgf3Y*(iUtA%ba`-Pu?^n-fFP~dpvnj-AyBKaJW)+{-ce}5$#DguerIN2
24rbxWpi{YTdJ&3iT??W6$?l#{@A?G8j--)v|TbGZq;_HaqHbhw}2&v0mUY=J6lL$MhcKn;XgI|zJmp*
01;Q@0XT>R0ssVVZ*FDSKfd5E4dt|K_z&S8_x<o;$Bma=F|FzDqP#$vGoEY#1aog~Wy=4LuCs~&piVX+
Q;&{e*II?7Wy=Z<NW)n^eyVI=$4I^-7b@t4MVjY>G@u4Q3HlB(d+LiLJm-R=h;`?dxC37Wb8ul}WgrA)
cw=lK261(7bY*iQ1ZZJ%Xd?z>Z)|K~awG?EWpZO>ZgeFHVQp|_a&uvBWF`t>aBp*Ta&K^GWhV$?a$#d@
Wpqp^2x4+!V{2t}QYi>wb97~LX>)5T1aNG1b1Ma7Z*6U1ECp?8Zgq1l17vS>E(LRJVRL9N1bSt1Z!iOI
Ze=k8ba!tu1$1a~Wo0u2W^Z+JGz4a8c4ajKb7^=s1#@L~Wo|bGWoc(<bT|ZVX>MgX1!He)Z*DpXb7gI5
LvL(vZaV~QWpi^p1!Zw{VQf7IXL4m>bY*fr2yt~~b98BMZa)HHbU*@MK|umvLP7#xLqh^zL_`8#MMVN%
Mn(c(M@Ir*NJs)-Nl5}<N=gD>OG^S@OiTh_O-%w{PEG<}Pfr40P*4J2QBeY4Qc?n6Q&R$8R8#_ARaF9C
R#pOES62dGSXcsISy=*KT3QNoaYAxoV{2t}Oj`+JVPk7kY+-X~Tnck>LULhaYh`p&T?J!da%FU025fI+
VRL9-2x4JlYjkO2YhVFkVF6-d0b^qUWMlzlWdUYp0cU3cXlMaxX#r|#0c&dkY-|B-Z2@j>0dH>saBu-}
aRG920dsQ!baVlAbpdvE0e5!+cz6MMc>#KQ31dQXVPk7$bWD2$aA|O5d<kPha$#d@Wpq+~1$1d_WMzI<
4VL$$c_gyK-vkb1V>yr3U)7OiEGa`mzoq#(6;V_O`>9xR8a*>q2D50xZ(4$R31H0PIsUw_;fcDKIn~;D
0000000000|Nj6000000TX!suv0v9m0LPL+_79KRH|I99{YEOV7tT#ZPWpkW1p!`O$dXTU&2q#dT$Zax
d1hGe8*-eZ2I646q$?$f9S>WJ$5c2n1xg(vzLX+s=TnOlIwpu5x<(TMczEDkZ1)BN1axJ1bQsH&ZxWNw
7!I9y+{RnQn@2DI{;m7<jj@=_gDCb(0R?SkWNBgGhp04`Gl!Y4#EOy;XgWfDE7LwMr|Y<=xPa<PwCjOf
p-EU><uvY*v*VyJx9`-=x0=4G6)zAUH(9jDAr2n^2welj7mcZoem^?%L*to!bRZoO^d~aUzM`;8jz95V
A_Ef(X>Md`c4>2IVr*pq1Y~7nX#oXeWo~q70tIbpY;0)*31nqsX-#QtY-t1vV`Xl1X-#QtY-t4rZE0h2
Zw3iuWn*bgX=8G42MS|lZggo)X=8G42n23nZf^+)WMyM%PGN3u3JGInZggo*VQy~=1aN6%Zwv@zWn*bj
X=85<31ek$bZJm&V{Z-xW@T-3Zx0D%Wn*bZWo>kC5DH^uZggozWo>kC5d>j$bZ-(~UdWP9bIo$ZB3zcM
M|oyg?;CQQqXyz&yre57i5(9G0)iue^mXv<w6)w(d6C|8kgcNIvvn*?253>L0b>G|!V30Z)+K@7h0D=S
BjVedlDqGVd368bwG2#j+mD9lQD0sr<;4X&8%0D>TgISeJ)kNFk^3<bZE>Xf*%skbRRcZ*dS!BNFavLH
WibPEcW*KUbZByAWite3Z*_7s1ZHV=Wi<nHX?QjTb7gjAZZ`#GX=iA3I0SBKZe=+FUqL|vUqV6xUqeFz
UqnO#UqwX%Uq(g(Uq?p*Ur0y-Ur9*<UrI^>UrS2@UrbB_UrkK{UrtT}Ur$d0Ur<m2Ur|v4Us6&6UsF>8
UsO~AUsY8CUshHEUsqQGUszZIUs+iKUs_rLVPOGcVgX}g0c2zWWn}?oW&vks0cdCeX=wp!Y5{9&0c>mm
ZEXQ=ZUJv^0dQ~uad821ashL50d#Z$b#(!Db^&*H0eE-;d3gbPdSj|16zH)>$g+grvznd|(VVK)`s##^
Jh_COk!RL4N(lR@SaKRYGgJn%Xv1$>f_VvG%;GuzyszPjx|liD+IRr~000000093000000004kq#k^Az
$U%@qU7>2Bz={du0O&G0&#r1CLJBFZ06hf(#5#SRxw8U!bIFfg*5RxK^~=*jK)}Ad3J<sl6abXef+K+R
b@1)9wcJs8k=}EVt)knrbu3H<Xi=&GV*-}K12h6(K|umvLP7#xLqh^zL_`8#MMVN%Mn(c(M@Ir*NJs)-
Nl5}<N=gD>OG^S@OiTh_O-%w{PEG<}Pfr40P*4J2QBeY4Qc?n6Q&R$8R8#_ARaF9CR#pOES62dGSXcsI
Sy=*KT3P{NVF6-d0b^qUWMlzlWdUYp0cU3cXlMaxX#r|#0c&dkY-|B-Z2@j>0dH>saBu-}aRG920dsQ!
baVlAbpdvE0e5!+cz6MMc>#KQh>TceV><CWmAB0rjf(Qx!Q2!JmvT|r)Y|jMQ5=qh1p!>4NmyOwH13hJ
<Df9N@6^q=n!c$OFAyI$S+vI?4j-^^(~tJj3|i&b2NlXOR2^DUyWY#wQk^*F-L`Td36}){9I$nc6v6<E
4*-nj($pTF88_k051ACjntmza&U>J{u?^n-fFP~dpvnj-AyBKaJW)+{-ce}5$#DguerIN21_K0id2nSM
uyu|U!T^j90F36+)E=H0H{s0>nH0sEektM3d!V}qb9G{Ld2nSf*z$T8ClZi8YCe|m_*?{lv>_T7tkE!8
{87}TyWT7ZV`yP=b7gcd*z$T8ClZi8YCe|m_*?{lv>_T7tkE!8{87}TyWT9nkIU)BIae{Jw9R4r0N>}O
)+shqImKG);D@8R3aUm3Jkg?^%&nV|dnPbniKwLeAs8?!PIJYq3V03Xs{mee0000000000KL7v#00000
#5#SRxw8U!bIFfg*5RxK^~=*jK)}Ad3J<sl6abXe1p-LEBNr;@ghiU?gEXK9KMDE{F?;HZBRuDVqlk6q
mbmq;7a>H<(%oY0=TGqa6l5KfYJkC@$v(fAa&d%-e7ws4kFK+d0H97bAybczVb@xPq-Dzr4oJgUK7Oif
U&jRjKPz&##IG7-47St%2#c>Z5R>jkTb_MKDq#SE<Vn}$%))Y#k9jx)*&ki4jZw^)YeO<P50WJJ$H7re
c8<G{1p@gO2n5}(1bO(?uXL+B(gNn{L2}utxi<$D8ry%w457b|%jv~AS23ov&0+fh-{+;)DK=9%#aim%
hoiX)sz%rRzT+P7s9j%|JK7ryjGlh`%f(kEcQ`Dp)4jAIiU9@$26Sm-Yh`j<cPx&vU)M(f$C5$z50Bb6
=QgwbMk=ru&P_#5`hlthZeeX@fL_JCQxeEQkVIXfYN5c23F83hGCI$$Y9m4lDXjoK2V`Y*VQFl0MYn(@
h5^MUvO8NyVMYp&P~kr{`@Vw(r~naH<N-K{32<^{V`+0~Z*E-!#21aJj($Hn^F!mAeRLol5%ecA&%UCt
OO8MBUn1B)zThtn<+N=058)p7{qSMOjh9_9t?BNfyg->Vo@@XB+#WAdR)2C|*D$SwhJOvD$-0{Gfin@`
<nKN_N?|2P1pz~<f5rCoWKZRyu3j3ckV4Jthm=C&OmaW<f`73y-iqLds5F){hncU$ijom%IzoLb(>^Yz
>$s@6fa*%L>wyFU00eGtZe`d%zThtn<+N=058)p7{qSMOjh9_9t?BNfyg->Vo@@aGb8l^B+#WAdR)2C|
*D$SwhJOvD$-0{Gfin@`<nKN_N?|2P^{p2nM9k9NV(jNn@cR^G9g}K+!Jx@Lzn5}xgo%8-2uQvo7b@t4
MVjY>G@u4Q3HlB(d+LiLJm-R=h;`?dxBvhE0000004D$d000000QnaP1l_I#dHB_@bgMhk0_N&La@nc5
HwP6O+keCip#vHLVPOGcVgX}g0c2zWWn}?oW&vks0cdCeX=wp!Y5{9&0c>mmZEXQ=ZUJv^0dQ~uad821
ashL50d#Z$b#(!Db^&*H0eE-;d3gbPdi$wZavD7|R0gwX!*5!Gc?n?5;yM1jui=Thm^szjcmV+b0|P-!
RR}^*L`g?QQ&a;|M?xV03jhEB(4Y?i2MYiJ01F5J01E*E0La=00XZ-L(V!0j2Lu2B0RR910000

-----END RGB CONSIGNMENT-----
```

You can also add a filename as a second argument, and the contract will be
saved into this file in binary form.


[kit]: https://rgbex.io/v1/schemata/RDYhMTR!9gv8Y2GLv9UNBEK1hcrCmdLDFk9Qd5fnO8k?name=NonInflatibleAsset.rgb
