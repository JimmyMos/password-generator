<script>
    import { onMount }     from 'svelte';
    import { browser }     from '$app/environment';
    //
    import { copyTextToClipboard } from "@jimmymos/dom-copy-helper";
    import { hasClass, addClass, removeClass, getParentsList, findParent } from "@jimmymos/dom-helper"
    //
    import Signature from "$lib/signature.svelte";
    //
    var maxLength     = 128;
    var minLength     = 6;
    var defaultLength = 24;
    var pwdLength     = defaultLength;
    var pwdSecurity   = setPwdSecurityLvl(pwdLength);
    var pwdSecurityContent = [
        {
            "lvl" : 'c1',
            "icon": '🚨',
            "txt" : 'This password is weak (hack time is less than 1 day)'
        },
        {
            "lvl" : 'c2',
            "icon": '👷',
            "txt" : 'This password have a medium security level (hack time is less than 10 years)'
        },
        {
            "lvl" : 'c3',
            "icon": '🔒',
            "txt" : 'Strong password (hack time is estimated to 24 trillons years)'
        },
        {
            "lvl" : 'c4',
            "icon": '👑',
            "txt" : 'Super strong password (no hack time estimated)'
        },
    ]
    //
    function setPwdSecurityLvl(length){
        var security = 4;
        //
        if(length < 64) {security = 3}
        if(length < 18) {security = 2}
        if(length < 11) {security = 1}
        //
        return security
    }
    //
    onMount(() => {
        if(browser){
            var elPwdInput    = document.querySelector('#pwd');
            //
            var elLengthInput = document.querySelector('.inputs_cont input[type=number]');
            var elLengthRange = document.querySelector('.inputs_cont input[type=range]');
            //
            var elCopy   = document.querySelector('.icon_button.copy');
            var elReset  = document.querySelector('.icon_button.reset');
            //
            var elUppercase    = document.querySelector('#uppercase');
            var elLowercase    = document.querySelector('#lowercase');
            var elNumbers      = document.querySelector('#numbers');
            var elSymbols      = document.querySelector('#symbols'); 
            var checkboxesList = [elUppercase, elLowercase, elNumbers, elSymbols]
            var checkedOptions = numberOfCheckedOptions();
            //
            var elPwdCopiedToast = document.querySelector('.confirmation_toast');
            //
            var lowercases = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'];
            var uppercases = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'];
            var numbers    = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'];
            var symbols    = ['!', '@', '#', '$', '%', '^', '&', '*', '(', ')', '_', '+', '-', '=', '{', '}', '[', ']', '|', ';', ':', "'", '"', '<', '>', ',', '.', '?', '/'];
            //
            function generatePassword(input, options = {
                "pwdLength"     : elLengthInput.value,
                "withUppercase" : elUppercase.checked,
                "withLowercase" : elLowercase.checked,
                "withNumbers"   : elNumbers.checked,
                "withSymbols"   : elSymbols.checked
            }){
                var usablesCarac = [];
                if(options.withUppercase === true) {usablesCarac = usablesCarac.concat(uppercases)}
                if(options.withLowercase === true) {usablesCarac = usablesCarac.concat(lowercases)}
                if(options.withNumbers   === true) {usablesCarac = usablesCarac.concat(numbers)}
                if(options.withSymbols   === true) {usablesCarac = usablesCarac.concat(symbols)}
                // console.log(usablesCarac);
                //
                var result = '';
                for (let index = 0; index < options.pwdLength; index++) {
                    const randomElement = usablesCarac[Math.floor(Math.random() * usablesCarac.length)];
                    result += randomElement;
                }
                input.value = result;
                //
                console.log(input.value.length);
                pwdSecurity = setPwdSecurityLvl(input.value.length)
            }
            function numberOfCheckedOptions(){
                var values     = checkboxesList.map(e => e.checked);
                var trueValues = values.filter(e => e === true);
                return trueValues.length;
            }
            generatePassword(elPwdInput);
            //
            [elLengthInput, elLengthRange].forEach(el => {
                el.addEventListener('input', function(ev){
                    elLengthInput.value = ev.currentTarget.value;
                    elLengthRange.value = ev.currentTarget.value;
                    //
                    generatePassword(elPwdInput);
                })
            });
            checkboxesList.forEach(el => {
                el.addEventListener('change', function(ev){
                    // First check if one checkbox (or less is checked) to block user from unchecking the last one
                    if(checkedOptions < 2 && ev.currentTarget.checked === false){
                        ev.currentTarget.checked = true;
                    }else{
                        generatePassword(elPwdInput);
                    }
                    checkedOptions = numberOfCheckedOptions();
                })
            });
            elReset.addEventListener('click', ev => {
                generatePassword(elPwdInput);
            });
            elCopy.addEventListener('click', ev => {
                copyTextToClipboard(elPwdInput.value);
                if(hasClass(elPwdCopiedToast, 'hidden')){
                    removeClass(elPwdCopiedToast, 'hidden');
                    addClass(elPwdCopiedToast, 'visible');
                    setTimeout(() => {
                        removeClass(elPwdCopiedToast, 'visible');
                        addClass(elPwdCopiedToast, 'hidden');
                    }, 3000);
                }else /*If has class visible*/ {
                    setTimeout(() => {
                        removeClass(elPwdCopiedToast, 'visible');
                        addClass(elPwdCopiedToast, 'hidden');
                    }, 3000);
                }
            });
        }
    })
</script>

<svelte:head>
    <title>Free strong password generator 🔑</title>
    <link rel="stylesheet" href="/css/variables.css?11231">
    <link rel="stylesheet" href="/css/reset.css?11231">
    <link rel="stylesheet" href="/css/styles.css?11231">
</svelte:head>

<div class="forkme">
    <a href="https://github.com/Hashiramaa/password-generator" target="_blank"></a>
    <img src="/forkme.png" alt="">
</div>
<h1>Free strong password generator 🔑</h1>
<div class="main_container">
    <div class="pwd_cont">
        <input type="text" id="pwd" value="My cool password">
        <div class="icon_button copy">
            <svg class="svg"><use href="/icons/copy.svg#svg"></use></svg>
        </div>
        <div class="icon_button reset">
            <svg class="svg"><use href="/icons/reset.svg#svg"></use></svg>
        </div>
    </div>
    <div class="controls_cont">
        <h2>Customize your password</h2>
        <div class="controls_inner_cont">
            <div class="length_cont">
                <h3>Password length ({minLength} - {maxLength})</h3>
                <div class="inputs_cont">
                    <input type="number" min="{minLength}" max="{maxLength}" value="{defaultLength}">
                    <input type="range" min="{minLength}" max="{maxLength}" value="{defaultLength}" step="1">
                </div>
            </div>
            <div class="options_cont">
                <div>
                    <input type="checkbox" id="uppercase" checked/>
                    <label for="uppercase">With uppercase</label>
                </div>
                <div>
                    <input type="checkbox" id="lowercase" checked/>
                    <label for="lowercase">With lowercase</label>
                </div>
                <div>
                    <input type="checkbox" id="numbers" checked/>
                    <label for="numbers">With numbers</label>
                </div>
                <div>
                    <input type="checkbox" id="symbols" checked/>
                    <label for="symbols">With symbols</label>
                </div>
            </div>
            <div class="security_level {pwdSecurityContent[pwdSecurity - 1].lvl}">
                <span>{pwdSecurityContent[pwdSecurity - 1].icon}</span>
                <span>{pwdSecurityContent[pwdSecurity - 1].txt}</span>
            </div>
        </div>
    </div>
    <span class="description">"The generated passwords are randomized and are not sent to anyone anywhere."</span>
</div>
<footer>
    <Signature />
</footer>
<div class="confirmation_toast hidden">
    <span>Password copied successfuly, keep it safe 🔑</span>
</div>

<style lang="scss">
    h1{
        width: 100%;
        padding: 40px 12px;
        text-align: center;
        font-size: 52px;
        font-weight: 500;
    }
    .description{
        margin: 40px 12px;
        max-width: 800px;
        font-size: 24px;
        font-weight: 500;
        display: flex;
        text-align: center;
    }
    .main_container{
        //border: 1px solid #000;
        max-width: 800px;
        margin: 0 auto;
        border-radius: 4px;
        padding: 4px;
        .pwd_cont{
            display: flex;
            align-items: center;
            gap: 10px;
            margin-bottom: 12px;
            #pwd{
                width: calc(100% - 100px);
                font-weight: 500;
                font-size: 36px;
                padding: 4px 12px 8px 12px;
                box-shadow: 0 2px 8px rgba(0,0,0,.15);
                border-radius: 8px 8px 0 0; 
                &:focus-visible{
                    outline: unset;
                }
            }
            .icon_button{
                width: 40px;
                height: 40px;
                background-color: #fff;
                border-radius: 100%;
                box-shadow: 0 2px 8px rgba(0,0,0,.15);
                cursor: pointer;
                transition: 250ms;
                &:hover{
                    transform: scale(110%);
                    svg{
                        fill: #000
                    }
                }
                svg{
                    padding: 10px;
                    max-width: 100%;
                    max-height: 100%;
                    fill: #5b5b5b;
                    transition: 250ms;
                }
            }
        }
        .controls_cont{
            display: flex;
            flex-direction: column;
            gap: 10px;
            background-color: #fff;
            padding: 12px;
            border-radius: 0 0 8px 8px; 
            box-shadow: 0 2px 8px rgba(0,0,0,.15);
            h2{
                font-size: 22px;
                font-weight: 500;
                margin-bottom: 10px;
                padding-bottom: 8px;
                border-bottom: 1px solid rgb(230, 230, 230);
            }
            .controls_inner_cont{
                display: flex;
                .length_cont{
                    display: flex;
                    flex-direction: column;
                    margin-right: 60px;
                    h3{
                        font-size: 16px;
                        font-weight: 500;
                        margin-bottom: 6px;
                    }
                    .inputs_cont{
                        display: flex;
                        align-items: center;
                        input[type=number]{
                            box-shadow: inset 0 2px 8px rgba(0,0,0,.15);
                            width: 80px;
                            height: 60px;
                            padding: 12px 10px 12px 20px;
                            border-radius: 4px;
                            font-weight: 500;
                            font-size: 22px;
                            margin-right: 8px;
                            &:focus-visible{
                                outline: unset;
                            }
                        }
                    }
                }
                .options_cont{
                    display: flex;
                    flex-direction: column;
                    gap: 10px;
                    margin-right: 60px;
                    & > div{
                        display: flex;
                        align-items: center;
                        gap: 8px;
                        input{
                            cursor: pointer;
                            height: 20px;
                            width: 20px;
                        }
                        label{
                            font-size: 20px;
                            font-weight: 500;
                            line-height: 1;
                            cursor: pointer;
                        }
                    }
                }
                .security_level{
                    max-width: 200px;
                    width: 100%;
                    padding: 6px 10px 10px 10px;
                    border-radius: 8px;
                    &.c1{
                        border: 1px solid rgb(141, 14, 14);
                        background-color: #f8f2f2;
                    }
                    &.c2{
                        border: 1px solid rgb(233, 97, 6);
                        background-color: #f8f0e9;
                    }
                    &.c3{
                        border: 1px solid green;
                        background-color: #f9fcf9;
                    }
                    &.c4{
                        border: 2px solid rgb(1, 107, 1);
                        background-color: #e5f3e5;
                    }
                    span:nth-of-type(1){
                        font-size: 30px;
                        display: flex;
                        justify-content: center;
                    }
                    span:nth-of-type(2){
                        font-weight: 500;
                        text-align: center;
                        display: flex;
                    }
                }
            }
        }
    }
    .confirmation_toast{
        position: absolute;
        bottom: 0;
        right: 0;
        margin: 20px;
        border-radius: 8px;
        padding: 20px 10px;
        background-color: #e2f1e2;
        border: 2px solid green;
        &:global(.visible){
            animation-name: inFromRightAndFadeOut;
            animation-duration: 3000ms;
            animation-fill-mode: forwards;
        }
        span{
            font-weight: 500;
        }
    }
    footer{
        margin-top: auto;
        background-color: #fff;
        box-shadow: 0 2px 8px rgba(0,0,0,.15);
        display: flex;
        justify-content: center;
    }
    .forkme{
        position: absolute;
        top: 0;
        right: 0;
        width: 160px;
        height: 160px;
        img{
            width: 100%;
            height: 100%;
        }
        a{
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
            width: 100%;
            height: 100%;
        }
    }
</style>