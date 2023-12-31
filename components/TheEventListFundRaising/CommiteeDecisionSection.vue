<template>
  <div class="bg-Gray-b2 dark:bg-LightGray-b2 p-5 rounded-xl">
    <ul class="flex flex-col gap-6">
      <h5 class="text-Gray-b5 dark:text-LightGray-b5 text-xl font-semibold">
        Commitee Decision
      </h5>
      <li
        v-for="(item, index) in props.state?.members"
        :key="index"
        class="flex w-full justify-between items-center"
      >
        <div class="flex gap-2 items-center">
          <img
            class="h-12 w-12 rounded-full"
            :src="item.avatar ? item.avatar : '/no-avatar.webp'"
          />
          <div class="flex flex-col gap-1">
            <client-only>
              <h4
                class="lg:text-base text-sm text-Gray-b5 dark:text-LightGray-b5 flex gap-2 items-center"
              >
                <el-tooltip
                  :content="item.member_wallet_address"
                  placement="top"
                >
                  <span>{{ shortenAddress(item.member_wallet_address) }}</span>
                </el-tooltip>
                <el-tooltip
                  :content="item.isCopied ? 'Copied' : 'Copy'"
                  placement="top"
                >
                  <i
                    class="isax isax-copy text-2xl"
                    v-if="!item.isCopied"
                    @click="copyTextToClipboard(item)"
                  />
                  <i
                    class="isax isax-copy-success text-2xl text-green-800"
                    v-if="item.isCopied"
                    @click="copyTextToClipboard(item)"
                  />
                </el-tooltip>
              </h4>
            </client-only>
            <h4
              class="lg:text-sm text-xs text-Gray-b3 dark:text-LightGray-b3 line-clamp-1"
            >
              {{ item.feedback }}
            </h4>
          </div>
        </div>
        <div class="flex items-center gap-2">
          <template
            v-if="
              props.state?.members[index].member_wallet_address.toLowerCase() ==
              account.toLowerCase()
            "
          >
            <button @click="dialogVisible = true" class="text-orange-600">
              Vote
            </button>
          </template>
          <i
            v-if="item.has_voted"
            :class="`text-xl isax isax-${
              item.decision ? 'like-1 text-green-800' : 'dislike text-red-800'
            }`"
          />
        </div>
      </li>
    </ul>
    <client-only>
      <el-dialog v-model="dialogVisible" title="Vote" width="30%">
        <Vote />
        <template #footer>
          <span class="dialog-footer">
            <el-button
              class="py-2 bg-transparent border-Gray-b5 dark:border-LightGray-b5 font-normal border-2 px-4 rounded-md text-base text-Gray-b5 dark:text-LightGray-b5 text-center"
              @click="dialogVisible = false"
              >Cancel</el-button
            >
            <el-button
              type="primary"
              class="py-2 bg-Primary border-Primary px-4 border-2 rounded-md text-base text-Gray-b5 dark:text-LightGray-b5 text-center"
              @click="checkVoteValidation"
            >
              Vote
            </el-button>
          </span>
        </template>
      </el-dialog>
    </client-only>
  </div>
</template>

<script setup>
import { ref, h } from "vue";
import { ElNotification } from "element-plus";
import { shortenAddress } from "@/utils/shortenAddress";
import Vote from "@/components/TheEvents/event-detail/Vote.vue";
import { useCryptoStore } from "~/store/crypto";
import { storeToRefs } from "pinia";
import { useVote } from "@/store/vote";
import { useLoading } from "@/store/loading";

const props = defineProps({
  state: null,
  committeeId: {
    default: null,
  },
});
const { state } = props;
const dialogVisible = ref(false);
const voteState = useVote();
const cryptoStore = useCryptoStore();
const { getRecordDecision } = useCryptoStore();
const loading = useLoading();
const { account } = storeToRefs(cryptoStore);

//methods

const copyTextToClipboard = async (item) => {
  try {
    await navigator.clipboard.writeText(item.member_wallet_address);
    props.state.members.map((item) => {
      item.isCopied = false;
    });
    item.isCopied = true;
    console.log("Copying to clipboard was successful!");
    setTimeout(() => {
      item.isCopied = false;
    }, 5000);
  } catch (err) {
    console.error("Failed to copy text: ", err);
  }
};
const checkVoteValidation = () => {
  if (!voteState.state.comment) {
    ElNotification({
      title: "Error",
      message: h("i", "add comment for your vote"),
      type: "error",
    });
    return false;
  } else {
    handleVote();
  }
};
const handleVote = async () => {
  loading.isLoading = true;
  const response = await getRecordDecision(
    props.committeeId,
    voteState.state.vote,
    voteState.state.comment
  );
  loading.isLoading = false;
  dialogVisible.value = false;
};
</script>
