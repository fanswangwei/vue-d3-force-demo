<template>
  <div id="app">
    <h3>d3-force</h3>
    <svg width="960" height="600"></svg>
  </div>
</template>
<script>
import * as d3 from 'd3'
export default {
  name: '',
  data() {
    return {
      // 节点信息
      nodesData: [
        { name: 'Lillian', sex: 'F', data: { r: 50 } },
        { name: 'Gordon', sex: 'M', data: { r: 50 } },
        { name: 'Sylvester', sex: 'M', data: { r: 50 } },
        { name: 'Mary', sex: 'F', data: { r: 50 } },
        { name: 'Helen', sex: 'F', data: { r: 50 } },
        { name: 'Jamie', sex: 'M', data: { r: 50 } },
        { name: 'Jessie', sex: 'F' },
        { name: 'Ashton', sex: 'M' },
        { name: 'Duncan', sex: 'M' },
        { name: 'Evette', sex: 'F' },
        { name: 'Mauer', sex: 'M' },
        { name: 'Fray', sex: 'F' },
        { name: 'Duke', sex: 'M' },
        { name: 'Baron', sex: 'M' },
        { name: 'Infante', sex: 'M' },
        { name: 'Percy', sex: 'M' },
        { name: 'Cynthia', sex: 'F' },
      ],
      // 连线信息
      linksData: [
        { source: 'Sylvester', target: 'Gordon', type: 'A' },
        { source: 'Sylvester', target: 'Lillian', type: 'A' },
        { source: 'Sylvester', target: 'Mary', type: 'A' },
        { source: 'Sylvester', target: 'Jamie', type: 'A' },
        { source: 'Sylvester', target: 'Jessie', type: 'A' },
        { source: 'Sylvester', target: 'Helen', type: 'A' },
        { source: 'Helen', target: 'Gordon', type: 'A' },
        { source: 'Mary', target: 'Lillian', type: 'A' },
        { source: 'Ashton', target: 'Mary', type: 'A' },
        { source: 'Duncan', target: 'Jamie', type: 'A' },
        { source: 'Gordon', target: 'Jessie', type: 'A' },
        { source: 'Sylvester', target: 'Fray', type: 'E' },
        { source: 'Fray', target: 'Mauer', type: 'A' },
        { source: 'Fray', target: 'Cynthia', type: 'A' },
        { source: 'Fray', target: 'Percy', type: 'A' },
        { source: 'Percy', target: 'Cynthia', type: 'A' },
        { source: 'Infante', target: 'Duke', type: 'A' },
        { source: 'Duke', target: 'Gordon', type: 'A' },
        { source: 'Duke', target: 'Sylvester', type: 'A' },
        { source: 'Baron', target: 'Duke', type: 'A' },
        { source: 'Baron', target: 'Sylvester', type: 'E' },
        { source: 'Evette', target: 'Sylvester', type: 'E' },
        { source: 'Cynthia', target: 'Sylvester', type: 'E' },
        { source: 'Cynthia', target: 'Jamie', type: 'E' },
        { source: 'Mauer', target: 'Jessie', type: 'E' },
      ],
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      // 获取d3容器
      let svg = d3.select('svg')
      let width = +svg.attr('width')
      let height = +svg.attr('height')
      console.log(width)
      console.log(height)
      // 使用节点数据设置模拟器
      let simulation = d3.forceSimulation().nodes(this.nodesData)
      simulation
        // 整个实例中心
        .force('center', d3.forceCenter(width / 2, height / 2))
        // 引力
        .force('charge', d3.forceManyBody().strength(-20))
        // 碰撞力 防止节点重叠
        .force(
          'collide',
          d3
            .forceCollide()
            .radius(60)
            .iterations(2)
        )
      // 创建连线
      let linkForce = d3.forceLink(this.linksData).id((d) => {
        return d.name
      })
      simulation.force('links', linkForce)
      let link = svg
        .append('g')
        .attr('class', 'links')
        .selectAll('line')
        .data(this.linksData)
        .enter()
        .append('line')
        .attr('stroke-width', 2)
        .style('stroke', this.linkColor)

      // 在svg元素中绘制圆圈
      let node = svg
        .append('g')
        .attr('class', 'nodes')
        .selectAll('circle')
        .data(this.nodesData)
        .enter()
        .append('circle')
        .attr('r', this.circleSize)
        .attr('fill', this.circleColor)
        .call(
          d3
            .drag()
            .on('start', dragstarted)
            .on('drag', dragged)
            .on('end', dragended)
        )

      // 作出举动时需要更新节点位置
      simulation.on('tick', tickAction)
      // 在圆圈中添加数据

      //建立用来放在每个节点和对应文字的分组<g>
      var gs = svg
        .append('g')
        .attr('class', 'nodes-info')
        .selectAll('info')
        .data(this.nodesData)
        .enter()
        .append('g')
        .attr('transform', function(d, i) {
          // console.log(d,i)
          var cirX = d.x
          var cirY = d.y
          return 'translate(' + cirX + ',' + cirY + ')'
        })
        .call(
          d3
            .drag()
            .on('start', dragstarted)
            .on('drag', dragged)
            .on('end', dragended)
        )
      //节点描述
      // 图片
      gs.append('image')
        .attr(
          'xlink:href',
          'https://okr-cdn-k8s-local.worktrans.cn/80164128/shared/10/13/14/43/f91f25ef974c456ebb73ffd108c15f41.png'
        )
        .attr('width', 60)
        .attr('class', 'image')
        .attr('height', 60)
        .attr('x', -35)
        .attr('y', -30)
        .attr('dy', 10)
      //文字
      gs.append('text')
        /*.attr("x",-10)
        .attr("y",-20)
        .attr("dy",10)*/
        .attr('class', 'text')
        .attr('x', -25)
        .attr('y', 30)
        .attr('dy', 10)
        .text(function(d) {
          return d.name
        })
      gs.append('text')
        /*.attr("x",-10)
        .attr("y",-20)
        .attr("dy",10)*/
        .attr('class', 'text')
        .attr('x', -25)
        .attr('y', 50)
        .attr('dy', 10)
        .text(function(d) {
          return '1111111'
        })
      // 更新函数
      function tickAction() {
        node
          .attr('cx', (d) => {
            return d.x
          })
          .attr('cy', (d) => {
            return d.y
          })
        gs.attr('transform', function(d) {
          return 'translate(' + d.x + ',' + d.y + ')'
        })

        link
          .attr('x1', (d) => {
            return d.source.x
          })
          .attr('y1', (d) => {
            return d.source.y
          })
          .attr('x2', (d) => {
            return d.target.x
          })
          .attr('y2', (d) => {
            return d.target.y
          })
      }
      // 放大和缩小
      /**
       * 节点（开始）拖拽事件
       */
      function dragstarted(d) {
        if (!d3.event.active) simulation.alphaTarget(0.002).restart()
        d.fx = d.x
        d.fy = d.y
        tickAction(d)
      }

      /**
       * 节点拖拽事件
       */
      function dragged(d) {
        d.fx = d3.event.x
        d.fy = d3.event.y
        tickAction(d)
      }

      /**
       * 节点（结束）拖拽事件
       */
      function dragended(d) {
        if (!d3.event.active) simulation.alphaTarget(0)
        tickAction(d)
      }
    },
    // 生成圆圈大小
    circleSize(d) {
      if (d.data && d.data.r) {
        return d.data.r
      } else {
        return 30
      }
    },
    // 生成圆圈颜色
    circleColor(d) {
      if (d.sex === 'M') {
        return 'blue'
      } else {
        return 'pink'
      }
    },
    // 生成线的颜色
    linkColor(d) {
      if (d.type === 'A') {
        return 'green'
      } else {
        return 'red'
      }
    },
  },
}
</script>
<style lang="scss">
svg {
  border: 1px solid #ccc;
}
.links line {
  stroke: #999;
  stroke-opacity: 0.6;
}
.nodes circle {
  width: 50px;
  height: 50px;
  stroke: #fff;
  stroke-width: 1.5px;
}
.nodes-info {
  .image {
  }
}
</style>
