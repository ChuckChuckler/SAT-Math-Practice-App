<script lang="ts">
    let { a, b, c, d } = $props();
</script>

<div class="bg-blue-200 box-border p-[15px] w-[50vw]">
    <div class="grid grid-rows-4 h-[300px] gap-3">
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]">
            <div class="w-[18%] bg-white">A</div>
            <div class="w-[79%] bg-white">{a}</div>
        </div>
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]">
            <div class="w-[18%] bg-white">B</div>
            <div class="w-[79%] bg-white">{b}</div>
        </div>
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]">
            <div class="w-[18%] bg-white">C</div>
            <div class="w-[79%] bg-white">{c}</div>
        </div>
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]">
            <div class="w-[18%] bg-white">D</div>
            <div class="w-[79%] bg-white">{d}</div>
        </div>
    </div>
</div>