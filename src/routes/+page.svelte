<script lang="ts">
  import { superForm, defaults } from "sveltekit-superforms";
  import { zod, zodClient } from "sveltekit-superforms/adapters";
  import { z } from "zod";

  import * as Form from "$lib/components/ui/form/index.js";
  import { Input } from "$lib/components/ui/input/index.js";

  export const schema = z.object({
    file: z
      .instanceof(File, { message: "Please upload a file." })
      .refine((f) => f.size < 5_000_000, "Max 5 MB upload size."),
  });

  const form = superForm(defaults(zod(schema)), {
    SPA: true,
    validators: zodClient(schema),
    onUpdate({ form }) {
      const reader = new FileReader();
      reader.onload = (e) => {
        console.log(e.target?.result as string);
      };

      reader.readAsText(form.data.file);
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
            on:input={(e) => ($formData.file = e.currentTarget.files?.item(0))}
          />
        </Form.Control>
        <Form.FieldErrors />
      </Form.Field>

      <Form.Button disabled={$delayed} class="w-full">Submit</Form.Button>
    </div>
  </form>
</main>
