// ==UserScript==
// @name         New Userscript
// @namespace    http://tampermonkey.net/
// @version      2023-12-05
// @description  try to take over the world!
// @author       You
// @match        https://pvpoke.com/moves/*
// @icon         https://www.google.com/s2/favicons?sz=64&domain=pvpoke.com
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    const styleElement = document.createElement('style');
    const cssRules = `
    #overlay_moves_table {
    position: fixed;
    transform: translate(-50%, -50%);
    top: 50%;
    left: 50%;
    max-width: 90vw;
    max-height:90vh;
    z-index: 100;
    background: white;
    font-size: 10px;
    overflow:auto;
    }
    .move_cell{
    min-width: 7vh;
    }
    #overlay_moves_table table{
    border-collapse: collapse;
    }
    #overlay_moves_table td, #overlay_moves_table th{
    border: 1px solid #000; /* Set border for table cells */
      padding: 8px;
      text-align: center;
      align-content: center;
    }
    #overlay_moves_table th.sticky-corner{
    	position: sticky;
      	top: 0;
      	left: 0;
      	background-color: #e0e0e0;
      	z-index: 4;
    }
    #overlay_moves_table th.sticky-header {
      position: sticky;
      top: 0;
      background-color: #f2f2f2;
      z-index: 3;
    }
    #overlay_moves_table th.sticky-col {
      position: sticky;
      left: 0;
      background-color: #f2f2f2;
      z-index: 3;
    }

    `;

    styleElement.appendChild(document.createTextNode(cssRules));
    document.head.append(styleElement);

    var html = ``;
    var tableContainer = document.createElement('div');
    var energy_costs = [];
    var dpt_ranges = [];
    var damage_ranges = [];
    var table_visiblity = 'hidden'
    document.addEventListener('keydown', (_event)=>{
        if(_event.key === '`' && table_visiblity == 'hidden'){
            html = buildTable();
            tableContainer.innerHTML = html;
            document.body.appendChild(tableContainer);
            table_visiblity = 'shown';
        }
        else{
            tableContainer.remove();
            table_visiblity = 'hidden';
        }
    });

    function getModeSelect(){
        return document.body.querySelector('.mode-select')?.value;
    }

    function getMoves(){
        var mode_select = getModeSelect();
        var moves = {};
        var move_table = document.body.querySelectorAll('.move-table-container');
        if(move_table.length){
            var moves_list_element = move_table[0].querySelector('.sortable-table');
            var rows = moves_list_element.querySelectorAll('tr');
            var move_name = '';
            energy_costs = [];
            dpt_ranges = [];
            damage_ranges = [];
            rows.forEach(_row => {
                //Skip adding header rows and hidden moves
                if(_row.querySelectorAll('.label').length) return;
                if(/display\: none/.test(_row.getAttribute('style'))) return;
                var row_vals = _row.querySelectorAll('td');
                move_name = row_vals[0].textContent;
                if(mode_select == 'fast'){
                    moves[move_name] = {
                        'name'   : move_name,
                        'type'   : row_vals[1].textContent,
                        'damage' : parseFloat(row_vals[2].textContent),
                        'energy' : parseFloat(row_vals[3].textContent),
                        'turns'  : parseFloat(row_vals[4].textContent),
                        'dpt'    : parseFloat(row_vals[5].textContent),
                        'ept'    : parseFloat(row_vals[6].textContent),
                    };
                    if(!energy_costs.includes(moves[move_name].ept)) energy_costs.push(moves[move_name].ept);
                    if(!dpt_ranges.includes(moves[move_name].dpt)) dpt_ranges.push(moves[move_name].dpt);
                }else if(mode_select == 'charged'){
                    moves[move_name] = {
                        'name'   : row_vals[0].textContent,
                        'type'   : row_vals[1].textContent,
                        'damage' : parseFloat(row_vals[2].textContent),
                        'energy' : parseFloat(row_vals[3].textContent),
                        'dpe'    : parseFloat(row_vals[4].textContent),
                        'effects': row_vals[5].textContent,
                    };
                    if(!energy_costs.includes(moves[move_name].energy)) energy_costs.push(moves[move_name].energy);
                    if(!damage_ranges.includes(moves[move_name].damage)) damage_ranges.push(moves[move_name].damage);
                }
            });
            //sort
            if(mode_select == 'fast') energy_costs.sort((a, b) => b - a);
            if(mode_select == 'fast') dpt_ranges.sort((a, b) => a - b);
            if(mode_select == 'charged') energy_costs.sort((a, b) => a - b);
            if(mode_select == 'charged') damage_ranges.sort((a, b) => a - b);
            return moves;
        }
    }

    function buildTable(){
        var mode_select = getModeSelect();
        var moves = getMoves();
        var html = ``;
        if(mode_select == 'fast'){
            html = `
            <div id="overlay_moves_table">
                <table>
                    <tr>
                        <th class="move_cell sticky-corner">DamagePerTurn<br>-----<br>EnergyPerTurn</th>`;

            dpt_ranges.forEach((_dpt)=>{
                html += `
                <th class="move_cell sticky-header">${_dpt}</th>`;
            });
            html+=`</tr>`;
            energy_costs.forEach(_energy=>{
                html+=`<tr><th class="move_cell sticky-col">${_energy}</th>`;
                for(let j=0; j<dpt_ranges.length; j++){
                    html+=`<td class="move_cell">`
                    for (let _move_name in moves){
                        var moveObj = moves[_move_name];
                        if(moveObj.dpt == dpt_ranges[j] &&
                           moveObj.ept == _energy &&
                           moves[_move_name]){
                            html += `<span class="type ${moveObj.type}" title="${getMoveTooltip(moveObj)}">${_move_name}(${moveObj.turns})</span><br>`;
                        }
                    }
                    html+=`</td>`
                }
            });


            html+= `
                </table>
            </div>
            `;
        }else if(mode_select == 'charged'){
            html = `
            <div id="overlay_moves_table">
                <table>
                    <tr>
                        <th class="move_cell sticky-corner">Damage<br>-----<br>Energy</th>`;

            damage_ranges.forEach((_dmg)=>{
                html += `
                <th class="move_cell sticky-header">${_dmg}</th>`;
            });
            html+=`</tr>`;
            energy_costs.forEach(_energy=>{
                html+=`<tr><th class="move_cell sticky-col">${_energy}</th>`;
                for(let j=0; j<damage_ranges.length; j++){
                    html+=`<td class="move_cell">`
                    for (let _move_name in moves){
                        var moveObj = moves[_move_name];
                        if(moveObj.damage == damage_ranges[j] &&
                           moveObj.energy == _energy &&
                           moves[_move_name]){
                            html += `<span class="type ${moveObj.type}" title="${getMoveTooltip(moveObj)}">${_move_name}${moveObj.effects ? '<b>*</b>' : ''}</span><br>`;
                        }
                    }
                    html+=`</td>`
                }
            });


            html+= `
                </table>
            </div>
            `;
        }
        console.log('',moves);
        return html;
    }

    function getMoveTooltip(_moveObj){
        var mode_select = getModeSelect();
        if(mode_select == 'fast'){
            return `${_moveObj.name}(${_moveObj.turns})\nDMG: ${_moveObj.damage}\nNRG: ${_moveObj.energy}\nDPT: ${_moveObj.dpt}\nEPT: ${_moveObj.ept}`
        }else if(mode_select == 'charged'){
            return `${_moveObj.name}\nDMG: ${_moveObj.damage}\nNRG: ${_moveObj.energy}\nDPE: ${_moveObj.dpe}${_moveObj.effects ? `\nEFFECTS: ${_moveObj.effects}` : ''}`
        }
    }

    // Your code here...
})();
