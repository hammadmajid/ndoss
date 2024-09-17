<script lang="ts">
  import { superForm, defaults } from "sveltekit-superforms";
  import { zod, zodClient } from "sveltekit-superforms/adapters";
  import { z } from "zod";

  import * as Form from "$lib/components/ui/form/index.js";
  import { Input } from "$lib/components/ui/input/index.js";
  import * as Table from "$lib/components/ui/table";
  import * as Accordion from "$lib/components/ui/accordion";

  import { createWorker } from "tesseract.js";

  export const schema = z.object({
    file: z
      .instanceof(File, { message: "Please upload a file." })
      .refine((f) => f.size < 5_000_000, "Max 5 MB upload size."),
  });

  let rawScannedText: string;
  let regNumbers: number[];

  const form = superForm(defaults(zod(schema)), {
    SPA: true,
    validators: zodClient(schema),
    async onUpdate({ form }) {
      let worker = await createWorker("eng");

      const {
        data: { text },
      } = await worker.recognize(form.data.file);

      rawScannedText = text;

      regNumbers = rawScannedText
        .split("\n")
        .map((x) => Number(x))
        .sort((a, b) => a - b);

      worker.terminate();
    },
  });

  const { form: formData, enhance, message, delayed } = form;
</script>

<main class="space-y-6 px-4 mt-8">
  <div class="space-y-2">
    <h1 class="scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl">
      NDOSS
    </h1>
    <p class="leading-7 text-muted-foreground [&:not(:first-child)]:mt-6">
      Niche Document OCR Scanner and Sorter
    </p>
  </div>

  <form
    method="POST"
    enctype="multipart/form-data"
    class="space-y-8"
    use:enhance
  >
    {#if $message}
      <p>{message}</p>
    {/if}

    <div class="space-y-2">
      <Form.Field {form} name="file">
        <Form.Control let:attrs>
          <Form.Label class="sr-only">Select file</Form.Label>
          <Input
            {...attrs}
            type="file"
            accept="image/*"
            disabled={$delayed}
            on:input={(e) => ($formData.file = e.currentTarget.files?.item(0))}
          />
        </Form.Control>
        <Form.FieldErrors />
      </Form.Field>

      <Form.Button disabled={$delayed} class="w-full">
        <span class="flex items-center justify-center gap-2">
          {#if $delayed}
            Loading
            <img
              src="/spinner.svg"
              alt="Loading spinner"
              class="inline animate-spin w-4"
            />
          {:else}
            Submit
          {/if}
        </span>
      </Form.Button>
    </div>
  </form>

  <div class="text-center space-y-4">
    {#if regNumbers}
      <Accordion.Root>
        <Accordion.Item value="item-1">
          <Accordion.Trigger>Show raw scanned text</Accordion.Trigger>
          <Accordion.Content>
            {rawScannedText}
          </Accordion.Content>
        </Accordion.Item>
      </Accordion.Root>
      <Table.Root>
        <Table.Header>
          <Table.Row>
            <Table.Head class="w-[100px] text-center"
              >Registration number</Table.Head
            >
            <Table.Head class="w-[100px] text-center">Student's name</Table.Head
            >
          </Table.Row>
        </Table.Header>
        <Table.Body>
          {#each regNumbers as number}
            <Table.Row>
              <Table.Cell>{number}</Table.Cell>
              <!-- TODO: map each registration number to name -->
              <Table.Cell>Unknown</Table.Cell>
            </Table.Row>
          {/each}
        </Table.Body>
      </Table.Root>
    {/if}
  </div>
</main>
