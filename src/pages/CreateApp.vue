<template>
  <div class="create-app">
    <div class="row no-wrap justify-around q-px-md q-pt-md">
      <div v-mutation="handler1" @dragenter="onDragEnter" @dragleave="onDragLeave" @dragover="onDragOver" @drop="onDrop" class="menu-target rounded-borders">
        <div id="box1" draggable="true" @dragstart="onDragStart" class="box navy" />
        <div class="box green" >
          <q-btn label="button" color="white" text-color="black" draggable="true" @dragstart="onDragStart"  id="box2"/>
        </div>
        <div id="box3" draggable="true" @dragstart="onDragStart" class="box red" >
          <q-input  label="Input" outlined />
        </div>

        <div id="box4" draggable="true" @dragstart="onDragStart" class="box orange" />
        <div id="box5" draggable="true" @dragstart="onDragStart" class="box navy" />
        <div id="box6" draggable="true" @dragstart="onDragStart" class="box red" />
        <div id="box7" draggable="true" @dragstart="onDragStart" class="box green" />
        <div id="box8" draggable="true" @dragstart="onDragStart" class="box orange" />
      </div>

      <div v-mutation="handler2" @dragenter="onDragEnter" @dragleave="onDragLeave" @dragover="onDragOver" @drop="onDrop" class="drop-target rounded-borders" >
      </div>
      <context-menu @show="onShow" @hide="onHide" :context_target="context_target"></context-menu>

    </div>

    <div class="row justify-around items-start">
      <div class="col row justify-center q-pa-md">
        <div class="text-subtitle1">Mutation Info</div>
        <div v-html="JSON.stringify(status)"></div>
      </div>
    </div>
  </div>
</template>

<script>
import ContextMenu from 'src/components/ContextMenu.vue';

export default {
  components: {ContextMenu},
  data() {
    return {
      is_disable: false,
      status: {},
      last_id: 0,

      context_target: null,
    };
  },
  methods: {
    handler1(mutationRecords) {
      // this.status = {};
      for (const index in mutationRecords) {
        const record = mutationRecords[index];
        const info = {
          menu_target: true,
          type: record.type,
          nodes_added: record.addedNodes.length > 0 ? true : false,
          nodes_removed: record.removedNodes.length > 0 ? true : false,
          oldValue: record.oldValue,
        };
        this.status = info;
      }
    },

    handler2(mutationRecords) {
      //  this.status = {};
      for (const index in mutationRecords) {
        const record = mutationRecords[index];
        const info = {
          content_target: true,
          type: record.type,
          nodes_added: record.addedNodes.length > 0 ? true : false,
          nodes_removed: record.removedNodes.length > 0 ? true : false,
          oldValue: record.oldValue,
        };
        this.status = info;
      }
    },

    // store the id of the draggable element
    onDragStart(e, create_new_node = true) {
      // creating new element by default (when moving elements from default menu container) but we dont need to do this when draging elements on working panel
      e.dataTransfer.setData("text", e.target.id);
      e.dataTransfer.setData("create_new", create_new_node);
      e.dataTransfer.dropEffect = "move";
    },

    onDragEnter(e) {
      // don't drop on other draggables
      if (e.target.draggable !== true) {
        e.target.classList.add("drag-enter");
      }
    },

    onDragLeave(e) {
      e.target.classList.remove("drag-enter");
    },

    onDragOver(e) {
      e.preventDefault();
    },

    createNewElement(node) {
      let node_name = node?.nodeName?.toLowerCase();
      let new_node = document.createElement(node_name);
      if(node_name == 'div'){
        new_node.classList = ["box bordered created-node"];
      }
      else if(!!node && !!new_node) {
        new_node.classList = node.classList;
      }
      // new_node.classList.add('bg-white');
      // new_node.classList.add('resizable');

      let htmlArray = Array.from(node.children).map(el => el.outerHTML);
      let plainHtml = htmlArray.join('');

      new_node.insertAdjacentHTML('afterbegin', plainHtml);

      const attrs = node.getAttributeNames().reduce((acc, name) => { return {...acc, [name]: node.getAttribute(name)}; }, {}); // copy all attributes from parent
      Object.keys(attrs).forEach(key => node.setAttribute(key, attrs[key]));  //set attributes to created element
      // new_node.setAttribute("contenteditable", true); // add a few aditional attributes for editing content and dragging
      new_node.setAttribute("draggable", true);
      new_node.setAttribute("id", "node_" + this.last_id);

      if(new_node.getAttribute('id')){
        new_node.addEventListener("contextmenu", ($event) => {
          this.handleRightClick(new_node);
        });
      }

      new_node.addEventListener("dragstart", ($event) => {
        this.onDragStart($event, false);
      });

      this.last_id++;
      return new_node;
    },
    onDrop(e) {
      e.preventDefault();

      if (this.status.menu_target) {
        return; //don`t drop on menu_target draggables
      }

      // drop on other draggables
      if (e.target.draggable === true) {
        e.target.classList.add("fit-content");
      }

      const draggedId = e.dataTransfer.getData("text");
      const draggedEl = document.getElementById(draggedId);

      // check if original parent node
      if (draggedEl?.parentNode === e.target) {
        e.target.classList.remove("drag-enter");
        return;
      }
      // draggedEl.parentNode.removeChild(draggedEl);
      if (e.dataTransfer.getData("create_new") == "false") {
        // make the exchange from elements list to working section
        e.target.appendChild(draggedEl);
        e.target.classList.remove("drag-enter");
        return;
      } else {
        // create new element if we are in the working section
        let element = this.createNewElement(draggedEl);
        e.target.appendChild(element);
      }
    },
    handleRightClick(node){
      this.context_target = '#'+ node.getAttribute('id');
      node.classList.add('bg-red', 'shadow-1')
    },
    onShow($event){
      setTimeout(()=>{
        console.log($event.srcElement);
        if($event.srcElement){
          this.context_target = $event.srcElement.getAttribute('id');
        }
        return;
      })
    },
    onHide(){
     let element = document.getElementById(this.context_target);
     console.log(element);
     if(element){
      element.classList.remove('bg-red');
     }
    }
  },
};
</script>

<style lang="scss">
.create-app {
  .menu-target {
    max-height: 70vh;
    width: fit-content;
    min-width: 200px;
    background-color: gainsboro;
    display: grid;
    overflow-y: auto;
    padding: 10px;
    min-height: 400px;
  }

  .drop-target {
    max-height: 70vh;
    width: fit-content;
    min-width: 200px;
    background-color: gainsboro;
    overflow-y: auto;
    padding: 10px;
  }
  .drag-enter {
    outline-style: dashed;
  }
  .box {
    min-width: 100px;
    min-height: 100px;
    float: left;
    cursor: pointer;
    transition: 1s all ease-in-out;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 5px;
  }

  @media only screen and (max-width: 500px) {
    .drop-target {
      height: 200px;
      width: 100px;
      min-width: 100px;
      background-color: gainsboro;
    }
    .box {
      width: 50px;
      height: 50px;
    }
  }

  .box:nth-child(3) {
    clear: both;
  }
  .navy {
    background-color: navy;
  }
  .red {
    background-color: firebrick;
  }
  .green {
    background-color: darkgreen;
  }
  .orange {
    background-color: orange;
  }
  .fit-content {
    width: fit-content;
    height: fit-content;
    transition: 1s all ease-in-out;
    padding: 10px;
    display: flex;
    gap: 10px;
  }

  .bordered {
    border: 1px dashed lightgrey;
  }

  .bordered {
    border-color: grey;
  }

  .created-node {
    background: darkgrey;
  }

  .resizable{
    resize: both;
    overflow: auto;
  }
}
</style>