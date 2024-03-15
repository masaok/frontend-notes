### Using Axios to make PUT request to an Express REST API

Using Axios with Bearer Token auth:

```ts
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  // Example token - in a real app, retrieve this from your auth state, config, or storage
  const authToken = "your_auth_token_here";

  try {
    const response = await axios.post(
      `/update-shift/${shiftId}`,
      shiftDetails,
      {
        headers: {
          Authorization: `Bearer ${authToken}`,
        },
      }
    );

    setUpdateStatus("Success: " + response.data.message);
  } catch (error) {
    if (axios.isAxiosError(error)) {
      setUpdateStatus(
        "Error: " + (error.response?.data.message || error.message)
      );
    } else {
      setUpdateStatus("Error: An unexpected error occurred");
    }
  }
};
```

Using Axios (more stable):

```ts
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  try {
    const response = await axios.post(`/update-shift/${shiftId}`, shiftDetails);

    setUpdateStatus("Success: " + response.data.message);
  } catch (error) {
    if (axios.isAxiosError(error)) {
      setUpdateStatus(
        "Error: " + (error.response?.data.message || error.message)
      );
    } else {
      setUpdateStatus("Error: An unexpected error occurred");
    }
  }
};
```

Using Fetch:

```ts
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  try {
    const response = await fetch(`/update-shift/${shiftId}`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(shiftDetails),
    });

    if (response.ok) {
      const data = await response.json();
      setUpdateStatus("Success: " + data.message);
    } else {
      throw new Error("Failed to update shift");
    }
  } catch (error: any) {
    setUpdateStatus("Error: " + error.message);
  }
};
```

https://chat.openai.com/share/5c593cc9-5046-4c6c-9a2f-dcfe5de41b86
