<template>
<!-- 子元件 -->
    <div>
        <div ref="sideNav" class="bm-menu">
            <nav class="bm-item-list">
                <slot></slot>
            </nav>
            <!-- hidden 要不要 display:none -->
            <span class="bm-cross-button cross-style" @click="closeMenu" :class="{ hidden: !crossIcon }">
                <span v-for="(x, index) in 2" :key="x" class="bm-cross" :style="{ position: 'absolute', width: '3px', height: '14px',transform: index === 1 ? 'rotate(45deg)' : 'rotate(-45deg)'}">
                </span>
            </span>

            <div ref="bmBurgerButton" class="bm-burger-button" @click="openMenu" :class="{ hidden: !burgerIcon }">
            <span class="bm-burger-bars line-style" :style="{top:20 * (index * 2) + '%'}" v-for="(x, index) in 3" :key="index"></span>
            </div>
        </div>

        

    </div>
</template>

<script>
    export default {
      name: 'menubar',
      data() {
        return {
          isSideBarOpen: false,
          winWidth:1920,
          burgerIcon:true,
          crossIcon:false
          
        };
      },
      props: {
        isOpen: {
          type: Boolean,
          required: false
        },
        right: {
          type: Boolean,
          required: false
        },
        width: {
          type: [String],
          required: false,
          default: '300'
        },
        disableEsc: {
          type: Boolean,
          required: false
        },
        noOverlay: {
          // 不要覆蓋
          type: Boolean,
          required: false
        },
        onStateChange: {
          type: Function,
          required: false
        }
        // ,
        // burgerIcon: {
        //   type: Boolean,
        //   required: false,
        //   default: true
        // },
        // crossIcon: {
        //   // x icon
        //   type: Boolean,
        //   required: false,
        //   default: true
        // }
        ,
        disableOutsideClick: {
          type: Boolean,
          required: false,
          default: false
        },
        closeOnNavigation: {
          type: Boolean,
          required: false,
          default: false
        }
      },
      methods: {
        openMenu() {
          this.$emit('openMenu');
          this.isSideBarOpen = true;
          this.$refs.sideNav.style.width = (this.winWidth*0.15) +'px';

          if (!this.noOverlay) {
            // 讓螢幕其他部分變暗
            document.body.classList.add('bm-overlay');
          }
          if (this.right) {
            // 改成靠右側的欄位樣式
            this.$refs.sideNav.style.left = 'auto';
            this.$refs.sideNav.style.right = '0px';
          }
          this.crossIcon = !this.crossIcon;
          this.burgerIcon = !this.burgerIcon;

          // this.$nextTick(function() {
          //   this.$refs.sideNav.style.width = this.width
          //     ? this.width + 'px'
          //     : '300px';
          // });
        },

        closeMenu() {
          this.crossIcon = !this.crossIcon;
          this.burgerIcon = !this.burgerIcon;

          this.$emit('closeMenu');
          this.isSideBarOpen = false;
          document.body.classList.remove('bm-overlay');
          this.$refs.sideNav.style.width = (this.winWidth*0.05)+'px';
        },

        closeMenuOnEsc(e) {
          e = e || window.event;
          if (e.key === 'Escape' || e.keyCode === 27) {
            this.closeMenu();
          }
        },
        documentClick(e) {
          let element = this.$refs.bmBurgerButton;
          let target = null;
          if (e && e.target) {
            target = e.target;
          }

          if (
            element &&
            element !== target &&
            !element.contains(target) &&
            !this.hasClass(target,'bm-menu') &&
            this.isSideBarOpen &&
            !this.disableOutsideClick
          ) {
            this.closeMenu();
          } else if (
            element &&
            this.hasClass(target,'bm-menu') &&
            this.isSideBarOpen &&
            this.closeOnNavigation
          ) {
            this.closeMenu();
          }
        },
        hasClass(element, className) {
          do {
            if (element.classList && element.classList.contains(className)) {
              return true;
            }
            element = element.parentNode;
          } while (element);
          return false;
        },
      },

      mounted() {
        if (!this.disableEsc) {
          document.addEventListener('keyup', this.closeMenuOnEsc);
        }
      },
      created: function() {
        document.addEventListener('click', this.documentClick);
      },
      destroyed: function() {
        document.removeEventListener('keyup', this.closeMenuOnEsc);
        document.removeEventListener('click', this.documentClick);
      },
      watch: {
        isOpen: {
          deep: true,
          immediate: true,
          handler(newValue, oldValue) {
            this.$nextTick(() => {
              if (!oldValue && newValue) {
                this.openMenu();
              }
              if (oldValue && !newValue) {
                this.closeMenu();
              }
            });
          }
        },
        right: {
          deep: true,
          immediate: true,
          handler(oldValue, newValue) {
            if (oldValue) {
              this.$nextTick(() => {
                this.$refs.bmBurgerButton.style.left = 'auto';
                this.$refs.bmBurgerButton.style.right = '36px';
                this.$refs.sideNav.style.left = 'auto';
                this.$refs.sideNav.style.right = '0px';
                document.querySelector('.bm-burger-button').style.left = 'auto';
                document.querySelector('.bm-burger-button').style.right = '36px';
                document.querySelector('.bm-menu').style.left = 'auto';
                document.querySelector('.bm-menu').style.right = '0px';
                document.querySelector('.cross-style').style.right='250px';
              });
            }
            if (newValue) {
              if (
                this.$refs.bmBurgerButton.hasAttribute('style')
              ) {
                this.$refs.bmBurgerButton.removeAttribute('style');
                this.$refs.sideNav.style.right = 'auto';
                document
                  .querySelector('.bm-burger-button')
                  .removeAttribute('style');
                document.getElementById('sideNav').style.right = 'auto';
                document.querySelector('.cross-style').style.right='0px';
              }
            }
          }
        }
      }
    };
</script>

<style>
    html {
      height: 100%;
    }
    .bm-burger-button {
      position: absolute;
      width: 24px;
      height: 20px;
      left: 30%;
      top: 30px;
      cursor: pointer;
    }
    .bm-burger-button.hidden {
      display: none;
    }
    .bm-burger-bars {
      background-color: #373a47;
    }
    .line-style {
      position: absolute;
      height: 20%;
      left: 0;
      right: 0;
    }
    .cross-style {
      position: absolute;
      top: 12px;
      right: 2px;
      cursor: pointer;
    }
    .bm-cross {
      background: #bdc3c7;
    }
    .bm-cross-button {
      height: 24px;
      width: 24px;
    }
    .bm-cross-button.hidden {
      display: none;
    }
    .bm-menu {
      height: 100%; /* 100% Full-height */
      width: 288px; /* 0 width - change this with JavaScript */
      position: absolute; /* Stay in place */
      z-index: 1000; /* Stay on top */
      top: 2%;
      left: 5%;
      border-radius:15px;
      background-color: rgb(63, 63, 65); /* Black*/
      overflow-x: hidden; /* Disable horizontal scroll */
      padding-top: 60px; /* Place content 60px from the top */
      transition: 0.5s; /*0.5 second transition effect to slide in the sidenav*/
    }

    .bm-overlay {
      background: rgba(0, 0, 0, 0.3);
    }
    .bm-item-list {
      color: #b8b7ad;
      margin-left: 10%;
      font-size: 20px;
    }
    .bm-item-list > * {
      display: flex;
      text-decoration: none;
      padding: 0.7em;
    }
    .bm-item-list > * > span {
      margin-left: 10px;
      font-weight: 700;
      color: white;
    }
</style>

