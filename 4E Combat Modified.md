//attacktarget = function(msg) {
on("chat:message", function(msg) {
if(msg.type == "whisper" && msg.content.toLowerCase().indexOf('!attack') !== -1 || msg.type == "api" && msg.content.toLowerCase().indexOf('!attack') !== -1) {
var slice = msg.content.split(" ");
// slice 1 is target, slice 2 is source, slice 3 is defensetype, slice 4 is damage roll slice 5 is to hit value slice 6 is bonusdamage slice 7 is critdamage slice 8 is extra to hit from the attack(say it comes with +2 attack or something)
// definining the enemy stats based on the first person named after the !attack
var enemytokenid = slice[1];
var playertokenid = slice[2];
var targetdefensename = slice[3];
var rollname = slice[4];
var tohit = slice[5];
var damage = slice[6];
var critdamage = slice[7];
var tohitmod = parseInt(slice[8]);
slice[9] += "";
var attackname = slice[9];
slice[10] += "";
var halfmiss = slice[10];
slice[11] += "";
var damagetype = slice[11];
var powertype = slice[12];
powertype += "";
if(damagetype.search('undefined') >= 0){
damagetype = "normal";
};
 
var curPageID = findObjs({_type: "campaign"})[0].get("playerpageid");
var targetenemytoken = getObj("graphic", enemytokenid);
if(!targetenemytoken)
{
sendChat('ERROR', 'Cannot find a target token.');
return;
}
var enemyname = targetenemytoken.get("name");
if(!targetenemytoken.get("represents"))
{
var errormsg = "target has no character " + enemyname;
sendChat('ERROR', errormsg);
return;
}
var targetenemycharacter = getObj("character", targetenemytoken.get("represents"));
var ac = findObjs({ _type: 'attribute', name: 'AC', _characterid: targetenemycharacter.id }, {caseInsensitive: true})[0];
 
var fort = findObjs({ _type: 'attribute', name: 'Fort', _characterid: targetenemycharacter.id }, {caseInsensitive: true})[0];
var will = findObjs({ _type: 'attribute', name: 'Will', _characterid: targetenemycharacter.id }, {caseInsensitive: true})[0];
 
var attributehp = findObjs({ _type: 'attribute', name: 'HP', _characterid: targetenemycharacter.id }, {caseInsensitive: true})[0];
var reflex = findObjs({ _type: 'attribute', name: 'Reflex', _characterid: targetenemycharacter.id }, {caseInsensitive: true})[0];
if(!ac || !fort || !will || !reflex)
{
sendChat('ERROR', 'target has no defenses.');
return;
}
var targetplayertoken = findObjs({_type: "graphic", _subtype: "token", _pageid: curPageID, _id: playertokenid}, {caseInsensitive: true})[0];
var playername = targetplayertoken.get("name")
 
var targetplayercharacter = getObj("character", targetplayertoken.get("represents"));
var enemyhp = targetenemytoken.get("bar1_value");
var resistname = "res" + damagetype;
var resistnumber = findObjs({ _type: 'attribute', name: resistname, _characterid: targetenemycharacter.id }, {caseInsensitive: true})[0];
var totaldmgMsg = "";
var damageMsg = "";
var youhitMsg = "";
if(targetdefensename.indexOf('AC') >= 0){
var targetdefensenumber = parseInt(ac.get("current"))
};
if(targetdefensename.indexOf('ac') >= 0){
var targetdefensenumber = parseInt(ac.get("current"))
};
if(targetdefensename.indexOf('Fort') >= 0){
var targetdefensenumber = parseInt(fort.get("current"))
};
if(targetdefensename.indexOf('Reflex') >= 0){
var targetdefensenumber = parseInt(reflex.get("current"))
};
if(targetdefensename.indexOf('Will') >= 0){
var targetdefensenumber = parseInt(will.get("current"))
};
if(targetdefensename.indexOf('fort') >= 0){
var targetdefensenumber = parseInt(fort.get("current"))
};
if(targetdefensename.indexOf('reflex') >= 0){
var targetdefensenumber = parseInt(reflex.get("current"))
};
if(targetdefensename.indexOf('will') >= 0){
var targetdefensenumber = parseInt(will.get("current"))
};
var rollmsg = "/roll " + rollname;
sendChat(playername, rollmsg, function(ops) {
var roll = randomInteger(20);
var right = "right";
var extramessage = "</big>";
var extramessagedamage = "</big>";
var damageresult = JSON.parse(ops[0].content);
tohitmod = parseInt(tohitmod);
var damagemod = 0;
// checking for dailies
log(powertype);
log(slice[12]);
if(powertype.search('daily') >= 0){
var isdailyused = findObjs({ _type: 'attribute', name: attackname, _characterid: targetplayercharacter.id }, {caseInsensitive: true})[0];
log(isdailyused);
if(!isdailyused)
{
log("not a daily");
}
else
{
if(isdailyused.get('current') <= 0)
{
sendChat(playername, "Already used " + attackname + " today");
return;
}
else
{
isdailyused.set('current', (isdailyused.get('current') - 1))
}
}
};

// this whole section is campaign specific stuff, my characters use marks to denote several status effects and it looks for them here
if(targetplayertoken.get('status_yellow', true)){
if(enemyname.search("Shenivix Caexoth") < 0){
tohitmod = parseInt(tohitmod - 2);
extramessage += "<li>Shenivix's Mark: <B>-2</b></li>";
}
};
if(targetenemytoken.get('status_brown', true)){
tohitmod = parseInt(tohitmod - 2);
extramessage += "<li>Shield the Fallen: <B>-2</b></li>";
};

if(targetplayertoken.get('status_pink', true)){
tohitmod = parseInt(tohitmod - 5);
extramessage += "<li>Memory Hole: <B>-5</b></li>";
};

if(targetenemytoken.get('status_blue', true)){
if(targetdefensename.indexOf('AC') >= 0){
tohitmod = parseInt(tohitmod - 1);
extramessage += "<li>Priest's Shield: <B>-1</b></li>";
}
};
 
if(targetplayertoken.get('status_bolt-shield', true)){
if(enemyname.search("Rith") >= 0){
tohitmod = parseInt(tohitmod) + 2;
extramessage += "<li>Bloodsworn Second wind: <b>-+2</b></li>";
}
};
 
if(targetplayertoken.get('status_cobweb', true)){
tohitmod = parseInt(tohitmod) - 2;
extramessage += "<li>Attack Bonus: <b>-2</b></li>";
};
 
if(targetplayertoken.get('status_lightning-helix', true)){
tohitmod = parseInt(tohitmod) + 2;
extramessage += "<li>Battle Cleric's Lore: <b>+2</b></li>";
};
if(targetplayertoken.get('status_screaming', true)){
tohitmod = parseInt(tohitmod) + 1;
extramessage += "<li>Captain's Inspiring Refrain<b>+1</b></li>";
};
if(targetenemytoken.get('status_arrowed', true)){
tohitmod = parseInt(tohitmod) + 2;
extramessage += "<li>Combat Advantage: <b>+2</b></li>";
};
if(targetenemytoken.get('status_bolt-shield', true)){
tohitmod = parseInt(tohitmod) - 2;
extramessage += "<li>Second wind: <b>-2</b></li>";
};
if(targetenemytoken.get('status_white-tower', true)){
tohitmod = parseInt(tohitmod) - 2;
extramessage += "<li>Target has cover: <b>-2</b></li>";
};
if(targetenemytoken.get('status_aura', true)){
tohitmod = parseInt(tohitmod) - 5;
extramessage += "<li>Superior cover: <b>-5</b></li>";
};
if(targetenemytoken.get('status_edge-crack', true)){
tohitmod = parseInt(tohitmod) + 2;
extramessage += "<li>Hit <b>+2</b></li>";
};
if(targetplayertoken.get('status_back-pain', true)){
tohitmod = parseInt(tohitmod) - 2;
extramessage += "<li>Splintering Shot: <b>-2</b></li>";
};
if(targetplayertoken.get('status_pummeled', true)){
tohitmod = parseInt(tohitmod) - 1;
extramessage += "<li>Splintering Shot: <b>-1</b></li>";
};
if(targetenemytoken.get('status_fist', true)){
damagemod = parseInt(damagemod) + 2;
extramessagedamage += "<li>Damage <b>+2</b></li>";
};
 
if(!findObjs({ _type: 'attribute', name: resistname, _characterid: targetenemycharacter.id }, {caseInsensitive: true})[0]){
 
}else{
var damageresistancetotal = parseInt(resistnumber.get('current'));
}
 
if(damageresistancetotal>=1){
damagemod = parseInt(damagemod) - damageresistancetotal;
extramessagedamage += "<li>Damage Resistance<b> -" + damageresistancetotal + "</b></li>";
};
if(targetenemytoken.get('status_green', true)){
if(playername.indexOf('Rith') >= 0){
var quarrydamage = randomInteger(8);
damagemod += quarrydamage;
extramessagedamage += "<li>Rith's Quarry: <b>+" + quarrydamage + "</b></li>";
};
};
if(playername.indexOf('Splug') >= 0){
var quarrydamage = randomInteger(6);
damagemod += quarrydamage;
extramessagedamage += "<li>Splug's Determination: <b>+" + quarrydamage + "</b></li>";
};

if(targetplayertoken.get('status_fluffy-wing', true)){
var quarrydamage = 5;
damagemod += quarrydamage;
extramessagedamage += "<li>Augment of War: <b>+5 </b></li>";
};

var finalroll = 0;
var damageroll = damageresult.total;
var damagefinal = parseInt(damageroll) + parseInt(damage) + parseInt(damagemod);
var finalroll = (parseInt(roll) + parseInt(tohit) + parseInt(tohitmod));
if(finalroll > parseInt(targetdefensenumber)){
 
var currentMsg = "<div align=" + right + "><ul><li class=" + right + ">Attack: <b>" + roll + "</b> + " + tohit + "</li>";
var youhitMsg = "<li>You Hit <i>" + enemyname + "'s</i> <b>" + targetdefensename + "</b> for <b>" + finalroll + "</b|></li></ul></div>";
var damageMsg = "<div align=" + right + "><ul><li class=" + right + ">Damage (<i>" + rollname + "</i>): <b>" + damageroll + "</b> + " + damage + "</b></li>";
var totaldmgMsg = "<li> Hitting " + enemyname + " for<big><b> " + damagefinal + "</big></b></li></ul></div>";
if(parseInt(roll) == 20){
var damagefinal = parseInt(critdamage) + parseInt(damage) +parseInt(damagemod);
var damageMsg = "<div align=" + right + "><ul><li class=" + right + "><big>Crit!</big> (<i>" + rollname + "</i>): <b>" + critdamage + "</b> + " + damage + "</li>";
var totaldmgMsg = "<li> Hitting " + enemyname + " for<big><b> " + damagefinal + "</big></b></li></ul></div>";
}
var setenemyhp = parseInt(enemyhp) - parseInt(damagefinal);
targetenemytoken.set("bar1_value", parseInt(setenemyhp)); 
if(attackname.indexOf('WoAF') >= 0)
{
targetenemytoken.set('status_cobweb', true);
};
if(attackname.indexOf('MemoryHole') >= 0)
{
targetenemytoken.set('status_pink', true);
};
if(attackname.indexOf('guiding') >= 0)
{
targetenemytoken.set('status_edge-crack', true);
};
if(attackname.indexOf('Priests-Shield') >= 0)
{
targetplayertoken.set('status_blue', true);
};
if(attackname.indexOf('Dazed') >= 0)
{
targetenemytoken.set('status_sleepy', true);
};
if(attackname.indexOf('Hypnotic_Pulse') >= 0)
{
targetenemytoken.set('status_sleepy', true);
};
if(attackname.indexOf('Splintering-Shot') >= 0)
{
targetenemytoken.set('status_back-pain', true);
};
if(attackname.indexOf('CrushingTurmoil1') >= 0)
{
targetenemytoken.set('status_cobweb', true);
};
if(attackname.indexOf('DoT') >= 0)
{
targetenemytoken.set('status_purple', true);
};
if(attackname.indexOf('CA') >= 0)
{
targetenemytoken.set('status_arrowed', true);
};
if(targetplayertoken.get('status_fluffy-wing', true))
{
targetenemytoken.set('status_arrowed', true);
};
if(attackname.indexOf('vortex') >= 0)
{
targetenemytoken.set('status_sleepy', true);
targetenemytoken.set('status_arrowed', true);
targetenemytoken.set('status_fluffy-wing', true);
};
}
 else{
var currentMsg = "<div align=" + right + "><ul><li class=" + right + ">Attack: <b>" + roll + "</b> + " + tohit + "</li>";
var youhitMsg = "<li>You Miss <i>" + enemyname + "'s</i> <b>" + targetdefensename + "</b> for <b>" + finalroll + "</b|></li></ul></div>";
if(halfmiss.indexOf('halfmiss') >= 0)
{
damagefinal = parseInt(damagefinal)/2;
damagefinal = Math.round(damagefinal);
var setenemyhp = (parseInt(enemyhp) - parseInt(damagefinal));
targetenemytoken.set("bar1_value", parseInt(setenemyhp));
var damageMsg = "<div align=" + right + "><ul><li class=" + right + ">Damage (<i>" + rollname + "</i>): <b>" + damageroll + "</b> + " + damage + "</b></li>";
var totaldmgMsg = "<li>half due to miss<big><b> " + damagefinal + "</big></b></li></ul></div>";
if(attackname.indexOf('Splintering-Shot') >= 0)
{
targetenemytoken.set('status_pummeled', true);
};
}
else{
var totaldmgMsg = "<li>Would've hit for<big><b> " + damagefinal + "</big></b></li></ul></div>";
var damageMsg = "<div align=" + right + "><ul><li class=" + right + ">Damage (<i>" + rollname + "</i>): <b>" + damageroll + "</b> + " + damage + "</b></li>";
if(attackname.indexOf('Hypnotic_Pulse') >= 0)
{
targetenemytoken.set('status_sleepy', true);
};

}
};
 
 
 
var damagetypemessage = "<li>" + damagetype + " damage</li>"
var finaltohitmessage = currentMsg + extramessage + youhitMsg;
sendChat(playername, finaltohitmessage);
var finaldmgmessage = damageMsg + extramessagedamage + damagetypemessage + totaldmgMsg;
sendChat(playername, finaldmgmessage);

if(targetenemytoken.get("bar1_max") === "") return;
   if(targetenemytoken.get("bar1_value") <= targetenemytoken.get("bar1_max") / 2) {
        targetenemytoken.set({
              status_redmarker: true
        });
    }
    else{
        targetenemytoken.set({
            status_redmarker: false
        })
    }

    if(targetenemytoken.get("bar1_value") <= 0) {
      targetenemytoken.set({
         status_dead: true
      });
    }
    else {
      targetenemytoken.set({
        status_dead: false
      });
    };
 
});
 
 
 
};
});
