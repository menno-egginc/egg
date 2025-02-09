<template>
  <div class="mx-4 xl:mx-0 mt-6 mb-8">
    <div class="text-sm mb-4">
      <div class="flex items-center space-x-1 -ml-px">
        <img
          :src="iconURL(hasProPermit ? 'egginc/pro_permit.png' : 'egginc/free_permit.png', 128)"
          class="h-4 flex-shrink-0"
        />
        <span class="font-serif truncate">{{ nickname }}</span>
      </div>
      <div class="mt-0.5 mb-1.5">
        <div class="text-xs">
          Last synced to server:
          <span v-tippy="{ content: lastRefreshed.format('LLL') }" class="whitespace-nowrap">
            {{ lastRefreshedRelative }}
          </span>
        </div>
        <div class="flex">
          <div
            v-tippy="{
              content: `
              <p>The game, while active, saves to Egg, Inc.&rsquo;s server every couple of minutes if network condition allows.
              Other than soon after a fresh launch of the game, such server syncs are unpredictable from the user&rsquo;s point of view.
              <span class='text-blue-300'>You can force close then reopen the app to reasonably reliably trigger a sync</span>
              (search for &ldquo;iOS force close app&rdquo; or &ldquo;Android force close app&rdquo; if you need help).</p>

              <p>However, even after an app-initiated sync, it may take an unpredicatible amount of time
              (usually within a minute or so) for the game&rsquo;s server to serve the updated save through its API,
              which is then picked up by this tool. There is no solution other than clicking &ldquo;Load Player Data&rdquo;
              periodically until the fresh save shows up. Please do not refresh too fast, which is not helpful.</p>`,
              allowHTML: true,
            }"
            class="flex items-center space-x-1"
          >
            <span class="text-xs text-gray-500">Why is my save out of date?</span>
            <base-info />
          </div>
          <div class="flex-0"></div>
        </div>
      </div>
      <div class="flex items-center">
        <span class="mr-1">Current balance:</span>
        <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
        <span class="text-ticket">{{ ticketsBalance }}</span>
      </div>
      <div class="flex items-center">
        <span class="mr-1">Lifetime earned:</span>
        <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
        <span class="text-ticket">{{ ticketsEarned }}</span>
      </div>
      <div class="flex items-center">
        <span class="mr-1">Lifetime spent:</span>
        <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
        <span class="text-ticket">{{ ticketsSpent }}</span>
      </div>
      <div class="flex items-center">
        <span class="mr-1">Total pieces owned:</span>
        <span class="text-ticket">{{ totalPiecesOwned }}/{{ totalPieces }}</span>
      </div>
      <div class="flex items-center">
        <span class="mr-1">Total value of possessions:</span>
        <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
        <span class="text-ticket">{{ totalValueOwned }}</span>
      </div>
      <div class="flex items-center">
        <span class="mr-1">Cost to complete collection:</span>
        <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
        <span class="text-ticket">{{ costToPurchaseEverything }}</span>
      </div>
      <div class="mt-2 flex items-center space-x-2">
        <span>What to show:</span>
        <input
          id="show-owned"
          v-model="showOwned"
          type="checkbox"
          class="focus:ring-green-500 h-4 w-4 text-green-600 border-gray-300 rounded focus:ring-0 focus:ring-offset-0 hover:cursor-pointer"
        />
        <span>Owned</span>
        <input
          id="show-unowned"
          v-model="showUnowned"
          type="checkbox"
          class="focus:ring-green-500 h-4 w-4 text-green-600 border-gray-300 rounded focus:ring-0 focus:ring-offset-0 hover:cursor-pointer"
        />
        <span>Not owned</span>
      </div>
    </div>

    <div class="space-y-4">
      <div>
        <h2 class="text-sm font-medium text-blue-700">Sets</h2>
        <div class="text-sm mt-1">
          Price next to each set name is the total value of the set, before discount.
        </div>
        <div
          class="grid sm:grid-cols-[repeat(auto-fill,minmax(336px,1fr))] gap-y-3 mt-2 empty:mt-0"
        >
          <template v-for="set in shellSets" :key="set.id">
            <div
              v-if="(showOwned && set.complete) || (showUnowned && !set.complete)"
              class="flex text-sm"
              :class="{ 'opacity-40': !set.complete }"
            >
              <div
                class="flex-0 h-8 w-8 rounded-full mr-2.5"
                :class="knownShellSetIds.includes(set.id) ? `set-${set.id}` : 'set-unknown'"
              ></div>
              <div class="leading-snug -mt-px">
                <div class="flex items-center">
                  <div class="mr-1">{{ set.name }}</div>
                  <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
                  <div class="flex-0 text-ticket -ml-px">{{ set.totalPrice }}</div>
                  <sup v-if="set.isNew" class="flex-0 text-[.6rem] text-red-600 ml-0.5">NEW!</sup>
                </div>
                <div class="flex items-center text-gray-500">
                  <div class="text-ticket">{{ set.ownedComponents }}/{{ set.totalComponents }}</div>
                  <div>&nbsp;pieces owned</div>
                  <template v-if="set.ownedComponents < set.totalComponents">
                    <div>,&nbsp;</div>
                    <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
                    <div class="flex-0 text-ticket -ml-px">{{ set.priceToComplete }}</div>
                    <div>&nbsp;to complete</div>
                  </template>
                </div>
              </div>
            </div>
          </template>
        </div>
      </div>

      <div v-for="shellType in shellTypes" :key="shellType">
        <h2 class="text-sm font-medium text-blue-700">{{ shellTypeNames[shellType] }}</h2>
        <div class="grid grid-cols-[repeat(auto-fill,minmax(168px,1fr))] gap-y-1 mt-2 empty:mt-0">
          <template v-for="shell in shells[shellType]" :key="shell.id">
            <div
              v-if="(showOwned && shell.owned) || (showUnowned && !shell.owned)"
              class="flex items-center text-sm"
              :class="{ 'opacity-40': !shell.owned }"
            >
              <div
                v-if="shell.setId && shellSetIds.includes(shell.setId)"
                v-tippy="{ content: `This is a piece in the ${shell.setName ?? '?'} set.` }"
                class="flex-0 h-4 w-4 rounded-full mr-1.5"
                :class="
                  knownShellSetIds.includes(shell.setId) ? `set-${shell.setId}` : 'set-unknown'
                "
              ></div>
              <div class="truncate mr-1">{{ shell.name }}</div>
              <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
              <div class="flex-0 text-ticket -ml-px">{{ shell.price }}</div>
              <sup v-if="shell.isNew" class="flex-0 text-[.6rem] text-red-600 ml-0.5">NEW!</sup>
              <div class="flex-0 w-4"></div>
            </div>
          </template>
        </div>
      </div>

      <div
        v-for="section in [
          { title: 'Chickens', objects: chickens },
          { title: 'Hats', objects: hats },
        ]"
        :key="section.title"
      >
        <h2 class="text-sm font-medium text-blue-700">{{ section.title }}</h2>
        <div class="grid grid-cols-[repeat(auto-fill,minmax(168px,1fr))] gap-y-1 mt-2 empty:mt-0">
          <template v-for="obj in section.objects" :key="obj.id">
            <div
              v-if="(showOwned && obj.owned) || (showUnowned && !obj.owned)"
              class="flex items-center text-sm"
              :class="{ 'opacity-40': !obj.owned }"
            >
              <div class="truncate mr-1">{{ obj.name }}</div>
              <img :src="ticketImageURL" class="flex-0 h-4 w-4" />
              <div class="flex-0 text-ticket -ml-px">{{ obj.price }}</div>
              <sup v-if="obj.isNew" class="flex-0 text-[.6rem] text-red-600 ml-0.5">NEW!</sup>
              <div class="flex-0 w-4"></div>
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue';
import dayjs from 'dayjs';
import localizedFormat from 'dayjs/plugin/localizedFormat';
import relativeTime from 'dayjs/plugin/relativeTime';

import {
  ei,
  getLocalStorage,
  iconURL,
  requestConfig,
  requestFirstContact,
  setLocalStorage,
  UserBackupEmptyError,
} from 'lib';

import BaseInfo from 'ui/components/BaseInfo.vue';

import AssetType = ei.ShellSpec.AssetType;

dayjs.extend(localizedFormat);
dayjs.extend(relativeTime);

type ShellSet = {
  id: string;
  name: string;
  totalComponents: number;
  ownedComponents: number;
  complete: boolean;
  totalPrice: number;
  discount: number;
  priceToComplete: number;
  isNew: boolean;
  spec: ei.IShellSetSpec;
};

type Shell = {
  id: string;
  name: string;
  isPartOfSet: boolean;
  setId?: string;
  setName?: string;
  type: AssetType;
  price: number;
  owned: boolean;
  isNew: boolean;
  spec: ei.IShellSpec;
};

type ShellObject = {
  id: string;
  name: string;
  type: AssetType.CHICKEN | AssetType.HAT;
  price: number;
  owned: boolean;
  isNew: boolean;
  spec: ei.IShellObjectSpec;
};

type Chicken = ShellObject & { type: AssetType.CHICKEN };
type Hat = ShellObject & { type: AssetType.HAT };

const props = defineProps<{ playerId: string }>();

const SHOW_OWNED_LOCALSTORAGE_KEY = 'showOwned';
const SHOW_UNOWNED_LOCALSTORAGE_KEY = 'showUnowned';
const showOwned = ref(getLocalStorage(SHOW_OWNED_LOCALSTORAGE_KEY) !== 'false');
const showUnowned = ref(getLocalStorage(SHOW_UNOWNED_LOCALSTORAGE_KEY) !== 'false');
watch(showOwned, () => {
  setLocalStorage(SHOW_OWNED_LOCALSTORAGE_KEY, showOwned.value);
});
watch(showUnowned, () => {
  setLocalStorage(SHOW_UNOWNED_LOCALSTORAGE_KEY, showUnowned.value);
});

const lastRefreshed = ref(dayjs());
const now = ref(dayjs());
const lastRefreshedRelative = computed(() => now.value && lastRefreshed.value.fromNow());
let refreshIntervalId: ReturnType<typeof setInterval> | undefined;
onMounted(() => {
  refreshIntervalId = setInterval(() => {
    now.value = dayjs(); // Force recomputation of lastRefreshedRelative
  }, 10000);
});
onBeforeUnmount(() => {
  clearInterval(refreshIntervalId);
});

// This async component does not respond to playerId changes.
const save = await requestFirstContact(props.playerId);
if (!save.backup || !save.backup.game) {
  throw new UserBackupEmptyError(props.playerId);
}
const config = await requestConfig(props.playerId);
const shellConfigs = config.dlcCatalog?.shells;
if (!shellConfigs || shellConfigs.length === 0) {
  throw new Error(`no shells found in /ei/get_config response`);
}

const ticketImageURL = iconURL('egginc/icon_shell_script_colored.png', 64);

const nickname = save.backup.userName ?? '';
const hasProPermit = save.backup.game.permitLevel === 1;
const ticketsEarned = save.backup.game.shellScriptsEarned ?? 0;
const ticketsSpent = save.backup.game.shellScriptsSpent ?? 0;
const ticketsBalance = ticketsEarned - ticketsSpent;
lastRefreshed.value = dayjs(Math.min(save.backup.settings!.lastBackupTime! * 1000, Date.now()));

const ownedShellIds = new Set<string>();
const ownedShellSetIds = new Set<string>();
for (const s of save.backup.shellDb?.shellInventory ?? []) {
  if (!s.identifier) {
    console.warn(`shellInventory item has no id`);
    continue;
  }
  if (!s.owned) {
    console.warn(`shellInventory item ${s.identifier} not owned`);
    continue;
  }
  ownedShellIds.add(s.identifier);
}
for (const s of save.backup.shellDb?.shellSetInventory ?? []) {
  if (!s.identifier) {
    console.warn(`shellSetInventory item has no id`);
    continue;
  }
  if (!s.owned) {
    console.warn(`shellSetInventory item ${s.identifier} not owned`);
    continue;
  }
  ownedShellSetIds.add(s.identifier);
}

const knownShellSetIds = [
  'black_white',
  'blueorange',
  'craftsman',
  'future_white',
  'new_order',
  'pink',
  'woodyellow',
];
const shellSets: ShellSet[] = [];
const shellSetNames: Record<string, string> = {};
for (const set of config.dlcCatalog?.shellSets ?? []) {
  const id = set.identifier;
  if (!id) {
    console.warn(`shell set has no id`);
    continue;
  }
  const name = set.name;
  if (!name) {
    console.warn(`shell set ${id} has no name`);
    continue;
  }
  shellSetNames[id] = name;
  // elementSet is true for a bunch of silo "sets" which are of course not
  // actual sets. Exclude them.
  if (!set.elementSet) {
    const discount = set.discount ?? 0;
    const isNew = !!set.isNew;
    shellSets.push({
      id,
      name,
      totalComponents: 0,
      ownedComponents: 0,
      complete: false,
      totalPrice: 0,
      discount,
      priceToComplete: 0,
      isNew,
      spec: set,
    });
  }
}
const shellSetIds = shellSets.map(set => set.id);

const shellTypes = [
  AssetType.GROUND,
  AssetType.HARDSCAPE,
  AssetType.MAILBOX,
  AssetType.TROPHY_CASE,
  AssetType.DEPOT_1,
  AssetType.HATCHERY_EDIBLE,
  AssetType.LAB_1,
  AssetType.HOA_1,
  AssetType.MISSION_CONTROL_1,
  AssetType.FUEL_TANK_1,
  AssetType.HYPERLOOP,
  AssetType.COOP,
  AssetType.SILO_0_SMALL,
] as const;
type ShellType = typeof shellTypes[number];
const shellTypeNames: Record<ShellType, string> = {
  [AssetType.GROUND]: 'Ground',
  [AssetType.HARDSCAPE]: 'Hardscape',
  [AssetType.MAILBOX]: 'Mailbox',
  [AssetType.TROPHY_CASE]: 'Trophy Case',
  [AssetType.DEPOT_1]: 'Depot',
  [AssetType.HATCHERY_EDIBLE]: 'Hatchery',
  [AssetType.LAB_1]: 'Lab',
  [AssetType.HOA_1]: 'Hall of Artifacts',
  [AssetType.MISSION_CONTROL_1]: 'Mission Control',
  [AssetType.FUEL_TANK_1]: 'Fuel Tank',
  [AssetType.HYPERLOOP]: 'Hyperloop Station',
  [AssetType.COOP]: 'Hab',
  [AssetType.SILO_0_SMALL]: 'Silo',
};
const shells: Record<ShellType, Shell[]> = {
  [AssetType.GROUND]: [],
  [AssetType.HARDSCAPE]: [],
  [AssetType.MAILBOX]: [],
  [AssetType.TROPHY_CASE]: [],
  [AssetType.DEPOT_1]: [],
  [AssetType.HATCHERY_EDIBLE]: [],
  [AssetType.LAB_1]: [],
  [AssetType.HOA_1]: [],
  [AssetType.MISSION_CONTROL_1]: [],
  [AssetType.FUEL_TANK_1]: [],
  [AssetType.HYPERLOOP]: [],
  [AssetType.COOP]: [],
  [AssetType.SILO_0_SMALL]: [],
};

// Maps `<setId> <assetType>` to the shell id of the canonical manifestation,
// e.g. "black_white hatchery_edible" -> "ei_hatchery_edible_black_white".
const setIdAssetTypeToCanonicalShellId = new Map<string, string>();
const addToPriceOfCanonicalShell = (
  canonType: ShellType,
  shell: {
    id: string;
    setId?: string;
    type: AssetType;
    price: number;
  }
) => {
  const typeStr = AssetType[shell.type].toLowerCase();
  const canonTypeStr = AssetType[canonType].toLowerCase();
  // Note that doing type string replacement in the id is unreliable; e.g.
  // "ei_hatchery_rocketfuel_black_white" does not contain the auto-generated
  // type string "rocket_fuel". So we use the (setId, assetType) -> canonId map
  // whenever possible.
  const canonId =
    (shell.setId && setIdAssetTypeToCanonicalShellId.get(`${shell.setId} ${canonTypeStr}`)) ??
    shell.id.replace(typeStr, canonTypeStr);
  for (const s of shells[canonType]) {
    if (s.id === canonId) {
      s.price += shell.price;
      return;
    }
  }
  console.warn(
    `cannot add ${shell.price} to price of ${canonId} which is not found in shells of type ${AssetType[canonType]}`
  );
};

// Collect shells by type.

// Sort by asset type so that the canonical manifestation of bundle appears
// first, e.g. "ei_depot_1_black_white" appears before "ei_depot_2_black_white",
// "ei_depot_3_black_white", etc.
shellConfigs.sort((s1, s2) => {
  const type1 = s1.primaryPiece?.assetType ?? 0;
  const type2 = s2.primaryPiece?.assetType ?? 0;
  return type1 - type2;
});
for (const shellConfig of shellConfigs) {
  const id = shellConfig.identifier;
  if (!id) {
    console.warn(`shell has no id`);
    continue;
  }
  const setId = shellConfig.setIdentifier ?? undefined;
  // A shell with a setId may not be part of an actual set; it could be in an
  // "element set", e.g. a bundle of different manifestations of the same silo
  // shell.
  const isPartOfSet = setId !== undefined && shellSetIds.includes(setId);
  const setName = setId !== undefined ? shellSetNames[setId] : undefined;
  const name = shellConfig.name || setName || '?';
  if (name === '?') {
    console.warn(`shell ${id} has no name`);
  }
  const type = shellConfig.primaryPiece?.assetType;
  if (type == null) {
    console.warn(`no asset type for shell ${id}`);
    continue;
  }
  const typeStr = AssetType[type].toLowerCase();
  const price = shellConfig.price ?? 0;
  const owned = ownedShellIds.has(id) || (setId !== undefined && ownedShellSetIds.has(setId));
  const isNew = !!shellConfig.isNew;
  const shell: Shell = {
    id,
    name,
    isPartOfSet,
    setId,
    setName,
    type,
    price,
    owned,
    isNew,
    spec: shellConfig,
  };
  switch (type) {
    case AssetType.GROUND:
    case AssetType.HARDSCAPE:
    case AssetType.MAILBOX:
    case AssetType.TROPHY_CASE:
    case AssetType.DEPOT_1:
    case AssetType.HATCHERY_EDIBLE:
    case AssetType.LAB_1:
    case AssetType.HOA_1:
    case AssetType.MISSION_CONTROL_1:
    case AssetType.FUEL_TANK_1:
    case AssetType.HYPERLOOP:
    case AssetType.COOP:
    case AssetType.SILO_0_SMALL:
      shells[type].push(shell);
      if (setId) {
        setIdAssetTypeToCanonicalShellId.set(`${setId} ${typeStr}`, id);
      }
      break;

    case AssetType.DEPOT_2:
    case AssetType.DEPOT_3:
    case AssetType.DEPOT_4:
    case AssetType.DEPOT_5:
    case AssetType.DEPOT_6:
    case AssetType.DEPOT_7:
      addToPriceOfCanonicalShell(AssetType.DEPOT_1, shell);
      break;

    case AssetType.HATCHERY_SUPERFOOD:
    case AssetType.HATCHERY_MEDICAL:
    case AssetType.HATCHERY_ROCKET_FUEL:
    case AssetType.HATCHERY_SUPERMATERIAL:
    case AssetType.HATCHERY_FUSION:
    case AssetType.HATCHERY_QUANTUM:
    case AssetType.HATCHERY_IMMORTALITY:
    case AssetType.HATCHERY_TACHYON:
    case AssetType.HATCHERY_GRAVITON:
    case AssetType.HATCHERY_DILITHIUM:
    case AssetType.HATCHERY_PRODIGY:
    case AssetType.HATCHERY_TERRAFORM:
    case AssetType.HATCHERY_ANTIMATTER:
    case AssetType.HATCHERY_DARK_MATTER:
    case AssetType.HATCHERY_AI:
    case AssetType.HATCHERY_NEBULA:
    case AssetType.HATCHERY_UNIVERSE:
    case AssetType.HATCHERY_ENLIGHTENMENT:
    case AssetType.HATCHERY_CHOCOLATE:
    case AssetType.HATCHERY_EASTER:
    case AssetType.HATCHERY_WATERBALLOON:
    case AssetType.HATCHERY_FIREWORK:
    case AssetType.HATCHERY_PUMPKIN:
      addToPriceOfCanonicalShell(AssetType.HATCHERY_EDIBLE, shell);
      break;

    case AssetType.LAB_2:
    case AssetType.LAB_3:
    case AssetType.LAB_4:
    case AssetType.LAB_5:
    case AssetType.LAB_6:
      addToPriceOfCanonicalShell(AssetType.LAB_1, shell);
      break;

    case AssetType.HOA_2:
    case AssetType.HOA_3:
      addToPriceOfCanonicalShell(AssetType.HOA_1, shell);
      break;

    case AssetType.MISSION_CONTROL_2:
    case AssetType.MISSION_CONTROL_3:
      addToPriceOfCanonicalShell(AssetType.MISSION_CONTROL_1, shell);
      break;

    case AssetType.FUEL_TANK_2:
    case AssetType.FUEL_TANK_3:
    case AssetType.FUEL_TANK_4:
      addToPriceOfCanonicalShell(AssetType.FUEL_TANK_1, shell);
      break;

    case AssetType.SHACK:
    case AssetType.SUPER_SHACK:
    case AssetType.SHORT_HOUSE:
    case AssetType.THE_STANDARD:
    case AssetType.LONG_HOUSE:
    case AssetType.DOUBLE_DECKER:
    case AssetType.WAREHOUSE:
    case AssetType.CENTER:
    case AssetType.BUNKER:
    case AssetType.EGGKEA:
    case AssetType.HAB_1K:
    case AssetType.HANGAR:
    case AssetType.TOWER:
    case AssetType.HAB_10K:
    case AssetType.EGGTOPIA:
    case AssetType.MONOLITH:
    case AssetType.PLANET_PORTAL:
    case AssetType.CHICKEN_UNIVERSE:
      addToPriceOfCanonicalShell(AssetType.COOP, shell);
      break;

    case AssetType.SILO_0_MED:
    case AssetType.SILO_0_LARGE:
    case AssetType.SILO_1_SMALL:
    case AssetType.SILO_1_MED:
    case AssetType.SILO_1_LARGE:
      addToPriceOfCanonicalShell(AssetType.SILO_0_SMALL, shell);
      break;

    default:
      console.warn(`unhandled asset type ${AssetType[type]} for shell ${id}`);
      continue;
  }
}

// Calculate shell set total components and prices.
for (const shellType of shellTypes) {
  for (const shell of shells[shellType]) {
    const setId = shell.setId;
    if (setId) {
      for (const set of shellSets) {
        if (set.id === setId) {
          set.totalComponents++;
          set.totalPrice += shell.price;
          if (shell.owned) {
            set.ownedComponents++;
          } else {
            set.priceToComplete += shell.price;
          }
          break;
        }
      }
    }
  }
}
for (const set of shellSets) {
  set.complete = set.ownedComponents === set.totalComponents;
  set.priceToComplete = Math.round(set.priceToComplete * (1 - set.discount));
}

// Sort shell sets.
shellSets.sort((s1, s2) => {
  if ((s1.spec.requiredEop ?? 0) !== (s2.spec.requiredEop ?? 0)) {
    return (s1.spec.requiredEop ?? 0) - (s2.spec.requiredEop ?? 0);
  }
  if ((s1.spec.requiredSoulEggs ?? 0) !== (s2.spec.requiredSoulEggs ?? 0)) {
    return (s1.spec.requiredSoulEggs ?? 0) - (s2.spec.requiredSoulEggs ?? 0);
  }
  if (s1.totalPrice !== s2.totalPrice) {
    return s1.totalPrice - s2.totalPrice;
  }
  return 0;
});
const shellSetIdOrder = shellSets.map(set => set.id);

// Sort shells.
for (const shellType of shellTypes) {
  shells[shellType].sort((s1, s2) => {
    // Set pieces first.
    if (s1.isPartOfSet && s2.isPartOfSet) {
      const index1 = shellSetIdOrder.indexOf(s1.setId!);
      const index2 = shellSetIdOrder.indexOf(s2.setId!);
      if (index1 !== index2) {
        return index1 - index2;
      }
    } else if (s1.isPartOfSet && !s2.isPartOfSet) {
      return -1;
    } else if (!s1.isPartOfSet && s2.isPartOfSet) {
      return 1;
    }
    if ((s1.spec.requiredEop ?? 0) !== (s2.spec.requiredEop ?? 0)) {
      return (s1.spec.requiredEop ?? 0) - (s2.spec.requiredEop ?? 0);
    }
    if ((s1.spec.requiredSoulEggs ?? 0) !== (s2.spec.requiredSoulEggs ?? 0)) {
      return (s1.spec.requiredSoulEggs ?? 0) - (s2.spec.requiredSoulEggs ?? 0);
    }
    if (s1.price !== s2.price) {
      return s1.price - s2.price;
    }
    return 0;
  });
}

// Handle objects (chickens & hats).
const ownedObjectIds = new Set<string>();
for (const s of save.backup.shellDb?.shellObjectInventory ?? []) {
  if (!s.identifier) {
    console.warn(`shellObjectInventory item has no id`);
    continue;
  }
  if (!s.owned) {
    console.warn(`shellObjectInventory item ${s.identifier} not owned`);
    continue;
  }
  ownedObjectIds.add(s.identifier);
}

const objects: ShellObject[] = [];
for (const objectConfig of config.dlcCatalog?.shellObjects ?? []) {
  const id = objectConfig.identifier;
  if (!id) {
    console.warn(`shellObject has no id`);
    continue;
  }
  const name = objectConfig.name ?? '?';
  if (name === '?') {
    console.warn(`shellObject ${id} has no name`);
    continue;
  }
  const type = objectConfig.assetType ?? undefined;
  if (type === undefined) {
    console.warn(`shellObject ${id} has no type`);
    continue;
  }
  if (type !== AssetType.CHICKEN && type !== AssetType.HAT) {
    console.warn(`shellObject ${id} has unexpected type ${AssetType[type]}`);
    continue;
  }
  const price = objectConfig.price ?? 0;
  const owned = ownedObjectIds.has(id);
  const isNew = !!objectConfig.isNew;
  objects.push({
    id,
    name,
    type,
    price,
    owned,
    isNew,
    spec: objectConfig,
  });
}
objects.sort((o1, o2) => {
  if ((o1.spec.requiredEop ?? 0) !== (o2.spec.requiredEop ?? 0)) {
    return (o1.spec.requiredEop ?? 0) - (o2.spec.requiredEop ?? 0);
  }
  if ((o1.spec.requiredSoulEggs ?? 0) !== (o2.spec.requiredSoulEggs ?? 0)) {
    return (o1.spec.requiredSoulEggs ?? 0) - (o2.spec.requiredSoulEggs ?? 0);
  }
  if (o1.price !== o2.price) {
    return o1.price - o2.price;
  }
  return 0;
});
const chickens = objects.filter((o: ShellObject): o is Chicken => o.type === AssetType.CHICKEN);
const hats = objects.filter((o: ShellObject): o is Hat => o.type === AssetType.HAT);

function sum(arr: number[]) {
  return arr.reduce((acc, cur) => acc + cur, 0);
}

const shellsFlat = Object.values(shells).flat();
const totalPieces = shellsFlat.length + objects.length;
const totalPiecesOwned =
  shellsFlat.filter(s => s.owned).length + objects.filter(s => s.owned).length;
const totalValueOwned =
  sum(shellsFlat.filter(s => s.owned).map(s => s.price)) +
  sum(objects.filter(s => s.owned).map(s => s.price));
const costToPurchaseEverything =
  sum(shellSets.map(set => set.priceToComplete)) +
  sum(shellsFlat.filter(s => !s.owned && !s.isPartOfSet).map(s => s.price)) +
  sum(objects.filter(o => !o.owned).map(o => o.price));
</script>

<style scoped lang="postcss">
.set-black_white {
  @apply border border-gray-300;
  background: linear-gradient(to right, black 0%, black 50%, white 50%, white 100%);
}

.set-blueorange {
  background: linear-gradient(to right, #5ccbf9 0%, #5ccbf9 50%, #f09837 50%, #f09837 100%);
}

.set-craftsman {
  @apply border-[0.5px] border-gray-300;
  background: linear-gradient(
    to right,
    #aa857f 0%,
    #aa857f 25%,
    #f8cf7f 25%,
    #f8cf7f 50%,
    #fcf8ef 50%,
    #fcf8ef 75%,
    #a2b097 50%,
    #a2b097 75%
  );
}

.set-future_white {
  @apply border border-[#cacacc];
  background: linear-gradient(to right, #cacacc 0%, #cacacc 50%, white 50%, white 100%);
}

.set-new_order {
  background: linear-gradient(
    to right,
    #2f2f31 0%,
    #2f2f31 33.33%,
    #58585a 33.33%,
    #58585a 66.67%,
    #ea3a2d 66.67%,
    #ea3a2d 100%
  );
}

.set-pink {
  background: #f3b1f9;
}

.set-woodyellow {
  background: linear-gradient(to right, #645343 0%, #645343 50%, #e6c14c 50%, #e6c14c 100%);
}

.set-unknown {
  @apply relative bg-gray-300;
}

.set-unknown::before {
  content: '?';
  @apply absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-gray-800;
}
</style>
